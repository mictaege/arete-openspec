# Design

## Context

Today `ScreenshotExtension` (arete core) is the only code that calls JUnit's
`ExtensionContext.publishFile`, and the arete-gradle `AreteTestListener.fileEntryPublished`
only recognizes the `arete-screenshot.` filename prefix, storing one screenshot per step
uniqueId in `SpecificationPlan.screenshotFiles`. There is no public API for step code to attach
arbitrary files to the step currently executing. See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- Public `StepContext` API in arete core with `attach(name, mediaType, bytes)` usable from any
  step body, publishing under a distinct `arete-attach.` prefix.
- arete-gradle receives, copies, and renders those attachments next to the owning step.
- Zero changes to existing step method signatures (family spec: arete's public API is
  compatibility-sensitive).

**Non-Goals:**
- Screenshot behavior changes (stays on `arete-screenshot.`, single file per step).
- Attachments in the history store.
- Size limits or cleanup beyond the existing screenshot cleanup rules.

## Decisions

**D1 — Reaching the current step: ThreadLocal holder, not a method parameter.**
`StepContext.current()` reads a `ThreadLocal<ExtensionContext>` that arete's step-execution
wrapping sets around each step invocation and clears afterwards. Alternative considered: adding a
`StepContext` parameter to `@Step`/`@Given`/... methods — rejected because it changes public method
signatures and breaks backward compatibility. Alternative considered: `ExtensionContext.Store` on
the test context only — rejected because attachments must land on the *step's* uniqueId, and the
holder gives direct access to the step-level context. ThreadLocal is safe here because JUnit runs
steps on the executing thread.

**D2 — `StepContext.attach` delegates to `publishFile` with an `arete-attach.` prefix.**
`attach(name, mediaType, bytes)` calls `context.publishFile("arete-attach." + name, mediaType, ...)`.
Keeping the prefix distinct from `arete-screenshot.` lets arete-gradle route files without
touching screenshot handling, and lets users attach multiple files to one step (screenshots are
one-per-step by design).

**D3 — arete-gradle generalizes reception, keeps screenshots untouched.**
`AreteTestListener.fileEntryPublished` accepts both `arete-screenshot.` (existing path) and
`arete-attach.` (new path). Attachments are stored in `SpecificationPlan` as
`Map<uniqueId, List<Attachment>>` where `Attachment` holds the original name and the published
path — a list, because one step may attach several files.

**D4 — Report rendering reuses the existing per-step resource pipeline.**
`HtmlWriter.copyResources` copies each attachment into `BuildDir.specsDir` under
`<uniqueHash>-<name>` (hash prefix avoids collisions between steps); `spec.ftlh` renders a
download link with the existing `icon-file.ftlh` icon in the step's detail area, next to where
the screenshot icon appears.

## Risks / Trade-offs

- [ThreadLocal leaks if a step thread is pooled/reused] → clear the holder in a `finally` block
  around step execution; also clear on `afterEach`.
- [Filename collisions or path traversal via user-supplied `name`] → sanitize the name
  (strip path separators) before prefixing, in both arete core and when copying in arete-gradle.
- [Large attachments bloat the report dir] → out of scope per proposal; same trade-off as
  screenshots today.
- [API addition to arete core is a public API change] → purely additive, allowed by the family
  spec's backward-compatibility requirement.

## Migration Plan

No migration needed — purely additive. Existing screenshot flow is unchanged; consumers upgrade
arete and arete-gradle together (shared version per release).

## Open Questions

None blocking: the prefix, holder mechanism, and rendering approach are decided above. Naming of
the public `StepContext` accessor (`current()` vs `of(...)`) can be settled during implementation
without affecting specs or task breakdown.
