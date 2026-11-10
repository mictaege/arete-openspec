# Tasks

## 1. arete core: StepContext API

- [x] 1.1 Add public `StepContext` interface/class in `com.github.mictaege.arete` with
  `attach(name, mediaType, bytes)` and a `current()` accessor backed by a ThreadLocal
  `ExtensionContext` holder; verify `./gradlew test` in `~/projects/arete` passes with a new unit
  test attaching a file from a step and asserting `publishFile` was called with the
  `arete-attach.` prefix.
- [x] 1.2 Wire the holder into step execution (set/clear in a `finally` around each step
  invocation); verify with a test that a step attaching a file results in a JUnit file entry on
  the step's uniqueId, and that the holder is cleared after the step.
- [x] 1.3 Sanitize the attachment `name` (strip path separators) before prefixing; verify with a
  unit test that `attach("../evil.txt", ...)` publishes a flat, safe filename.

## 2. arete-gradle: reception, copying, rendering

- [x] 2.1 Generalize `AreteTestListener.fileEntryPublished` to also accept the `arete-attach.`
  prefix and register attachments in `SpecificationPlan` as a per-uniqueId list; verify
  `./gradlew test` in `~/projects/arete-gradle` passes with a listener test publishing an
  `arete-attach.` file entry.
- [x] 2.2 Extend `HtmlWriter.copyResources` to copy each attachment into `specs/` as
  `<uniqueHash>-<name>`; verify the copied file exists in the report output dir in a writer test.
- [x] 2.3 Render download links with the `icon-file` icon on the owning step in `spec.ftlh`;
  verify the generated HTML contains the attachment link.

## 3. End-to-end verification

- [x] 3.1 Add a demo journey step in `arete-gradle-test` that calls `StepContext.current()`
  `.attach(...)`; verify `./gradlew test` there produces a report HTML with the attachment link
  and the copied file under `specs/`.

## Workflow follow-up

- Add the same-named `step-context-attach` change to the `arete` and `arete-gradle` repo-local
  roots (per family spec, cross-repo features get a same-named change in each affected repo).
- Archive this change together with the repo-local ones at release time.
