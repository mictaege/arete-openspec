# Design: `NarrativeContext.attach`

## Context

`StepContext` (archived change `2026-11-10-step-context-attach`) lets step bodies attach dynamic
files via a `ThreadLocal<ExtensionContext>` held by `StepContextExtension` during
`interceptTestMethod`, published under the `arete-attach.` prefix. `@Narrative` has no equivalent:
its `attachmentResourcePath` only supports static classpath files. This change adds the
narrative-level counterpart. See proposal.md for motivation.

Current state observed in code:

- `arete`: `StepContext` / `StepContextExtension` are the template to mirror. Stereo-type
  annotations (`@Given`/`@When`/`@Then`/`@Step`/`@ItShould`) register
  `@ExtendWith(StepContextExtension.class)` directly; `@Narrative` currently has no `@ExtendWith`.
- `arete-gradle`: `AreteTestListener.fileEntryPublished` routes `arete-screenshot.` and
  `arete-attach.` prefixes into `SpecificationPlan` maps keyed by uniqueId;
  `SpecificationStep` exposes `attachments` / `hasAttachments`; `HtmlWriter.copyResources` copies
  step attachments as `<uniqueHash>-<name>` and `deleteOriginalAttachments` removes the published
  originals; `spec.ftlh` renders step attachment chips via the `attachmentChips` macro and static
  narrative attachments inside the `hasNarrative` block.

## Goals / Non-Goals

**Goals:**

- Public `NarrativeContext.current().attach(name, mediaType, bytes)` usable from class-level
  lifecycle methods (`@BeforeAll`, `@AfterAll`, test-class constructor) of any `@Narrative`-annotated
  class, publishing files under a distinct `arete-narrative-attach.` prefix.
- Zero changes to `@Narrative` attributes; purely additive public API on `arete` core.
- arete-gradle receives, stores, copies, and renders narrative attachments inside the narrative
  block of `spec.ftlh`, including when `attachmentResourcePath` is empty.

**Non-Goals:**

- Attaching to the enclosing narrative from inside a step/test method.
- Changes to step attachments, screenshots, or the history store.
- New `@Narrative` annotation elements.

## Decisions

### D1: Distinct prefix `arete-narrative-attach.` over reusing `arete-attach.`

Reusing `arete-attach.` would key files by class uniqueId into the existing step-attachment map,
rendering them both as step chips and (if also handled) in the narrative block — double rendering.
A distinct prefix keeps routing explicit: `AreteTestListener` can route to a separate map without
ambiguity. Constant `NarrativeContext.ATTACHMENT_NAME_PREFIX = "arete-narrative-attach"`, mirroring
`StepContext.ATTACHMENT_NAME_PREFIX`.

### D2: `NarrativeContextExtension` as `InvocationInterceptor` on the class-level lifecycle

`StepContextExtension` intercepts `interceptTestMethod`. The narrative equivalent must hold the
class-level `ExtensionContext` during `interceptBeforeAllMethod`, `interceptAfterAllMethod`, and
`interceptTestClassConstructor`, clearing the `ThreadLocal` in a `finally` so a failure cannot leak
state to the next class. This covers exactly the class-level lifecycle window chosen in the proposal.

### D3: Registration via meta-annotation `@ExtendWith(NarrativeContextExtension.class)` on `@Narrative`

Registering on `@Narrative` itself (rather than on all five stereo-type annotations) means
`NarrativeContext.current()` fails fast with `IllegalStateException` wherever no narrative exists —
the desired behavior. `@Narrative` stays a pure annotation plus one meta-annotation; all existing
attributes unchanged.

### D4: Separate narrative-attachment map in `SpecificationPlan`, keyed by uniqueId (list per key)

Mirrors the step-attachment structure: `narrativeAttachmentFiles: MutableMap<String, MutableList<StepAttachment>>`
with `addNarrativeAttachment(uniqueId, name, path)` / `narrativeAttachments(uniqueId)` /
`removeNarrativeAttachments(uniqueId)`. Exposed on `SpecificationStep` as `narrativeAttachments` /
`hasNarrativeAttachments`.

### D5: Copy naming `<uniqueHash>-<name>` for dynamic narrative attachments

Static narrative resources are copied under their plain file name; dynamic ones get the
`<uniqueHash>-` prefix to avoid collisions with static files of the same name and with step
attachments of the same node. `HtmlWriter.copyResources` copies narrative attachments to
`specs/<uniqueHash>-<name>`; `deleteOriginalNarrativeAttachments` removes the published originals
afterwards, mirroring `deleteOriginalAttachments`.

### D6: Rendering inside the existing `hasNarrative` block in `spec.ftlh`

A new `<#if step.hasNarrativeAttachments>` block inside the `step.hasNarrative` section renders
dynamic attachments in a `<div class="attachments">` (same CSS classes as static ones) with
`href="./<uniqueHash>-<name>"`, file icon + name. Gating on `hasNarrative` (not on
`narrative.hasAttachments`) means dynamic attachments render even when `attachmentResourcePath` is
empty.

## Risks / Trade-offs

- [Risk] `publishFile` on the class-level `ExtensionContext` from `@BeforeAll` — JUnit publishes the
  file against the class container, which is exactly the uniqueId the listener receives. → Verified
  pattern: screenshots already publish from extension contexts; the listener keys by uniqueId.
- [Risk] Thread safety of the `ThreadLocal` when `@BeforeAll`/`@AfterAll` run on a different thread
  than the constructor. → Same limitation as `StepContext`; documented behavior, `finally` cleanup
  prevents leakage.
- [Trade-off] Nested `@Scenario` classes attach to their own class context, landing on the scenario
  node rather than the outer spec. → Accepted for v1; matches the proposal's scoping.
- [Trade-off] No size limits or extra cleanup beyond existing attachment rules. → Consistent with
  step attachments; revisit only if report dir growth becomes an issue.

## Migration Plan

Not applicable — purely additive API and additive report rendering. Existing specs and reports are
unaffected; consumers upgrade `arete` and `arete-gradle` together per the family's shared-version rule.

## Open Questions

None blocking. (Attaching from inside a test method to the enclosing narrative is a known follow-up,
explicitly out of scope.)
