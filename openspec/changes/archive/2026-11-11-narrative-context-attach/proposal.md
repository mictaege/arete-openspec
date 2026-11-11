# Proposal: `NarrativeContext.attach` — attach dynamic content to the current narrative

Follow-up to the archived `2026-11-10-step-context-attach` change. Proposal-only — no design/tasks yet.

## Why

`@Narrative.attachmentResourcePath` only supports static classpath files. Content that is generated
during the run (aggregated results in `@AfterAll`, a manifest built at construction time, a report
produced by a build step) cannot be pinned to a narrative, even though it belongs there in the
report. `StepContext` solved this for steps; narratives need the equivalent on the declaration side.

## What Changes

- arete core: new public `NarrativeContext`, mirroring `StepContext`:
  - `static NarrativeContext current()` backed by a `ThreadLocal<ExtensionContext>`, throwing
    `IllegalStateException` when no narrative scope is active on the calling thread.
  - `attach(String name, MediaType mediaType, byte[] bytes)` publishing a JUnit file under the
    distinct `arete-narrative-attach.` prefix (constant `ATTACHMENT_NAME_PREFIX`) on the
    class-level `ExtensionContext` via `publishFile`; path separators in `name` sanitized to `_`
    exactly like `StepContext`.
- arete core: new `NarrativeContextExtension` (an `InvocationInterceptor`) that holds the
  class-level `ExtensionContext` for the duration of `interceptBeforeAllMethod`,
  `interceptAfterAllMethod` and `interceptTestClassConstructor`, clearing it afterwards even on
  failure. Registered by meta-annotating `@Narrative` itself with
  `@ExtendWith(NarrativeContextExtension.class)`, so the context exists exactly where a narrative
  is declared and nowhere else. `@Narrative` stays otherwise unchanged (purely additive public
  API, allowed by the family spec's backward-compatibility rule).
- Scope of `current()` is the class-level lifecycle only (`@BeforeAll`/`@AfterAll`/constructor).
  Nested classes attach to their own narrative: a `@Scenario`'s `beforeAll` publishes on the
  scenario's class context and lands on that scenario node. Attaching from inside a test method to
  the enclosing narrative is explicitly deferred.
- arete-gradle reception: `AreteTestListener.fileEntryPublished` accepts the
  `arete-narrative-attach.` prefix in addition to `arete-screenshot.`/`arete-attach.` and registers
  files in a new `SpecificationPlan` narrative-attachment map keyed by uniqueId (list per
  uniqueId, like step attachments); exposed as `SpecificationStep.narrativeAttachments` /
  `hasNarrativeAttachments`.
- arete-gradle rendering: `HtmlWriter.copyResources` copies narrative attachments into `specs/` as
  `<uniqueHash>-<name>` (hash prefix avoids collisions with static narrative resources, which use
  the plain file name) and deletes the original published files afterwards;
  `spec.ftlh` renders them inside the narrative's existing `<div class="attachments">` next to the
  `attachmentResourcePath` entries (href `./<uniqueHash>-<name>`, file icon + name), whenever the
  node `hasNarrative` — including when the annotation's `attachmentResourcePath` is empty.
- Delta specs when designed: arete repo — `narratives` spec extended or a new `narrative-context`
  capability mirroring `step-context`; arete-gradle repo — `narrative-rendering` and `attachments`
  specs extended.

### Decisions already made (explored and agreed)

- Distinct prefix `arete-narrative-attach.` rather than reusing `arete-attach.` — reusing would key
  by class uniqueId into the step-attachment map and render as a step chip as well as in the
  narrative block (double render); a separate map keeps routing explicit.
- Registration via `@ExtendWith` meta-annotation on `@Narrative`, not on all five stereo-type
  annotations — `current()` then fails fast where no narrative exists.
- Class-level lifecycle window only for v1.
- Copy naming `<uniqueHash>-<name>` for dynamic narrative attachments.

## Out of Scope

- Changes to the `@Narrative` annotation attributes (no new annotation element; static
  `attachmentResourcePath` behavior unchanged).
- Attaching to the enclosing narrative from inside a step/test method (possible follow-up).
- Step attachments (`arete-attach.`) and screenshots — unchanged.
- Narrative attachments in the history store.
- Size limits/cleanup beyond the existing attachment cleanup rules.

## Affected Repos

- `arete` (public `NarrativeContext` API + `NarrativeContextExtension` + `@ExtendWith` on `@Narrative`)
- `arete-gradle` (reception, plan storage, copying, rendering)
- `arete-gradle-test` (demo example, optional)
