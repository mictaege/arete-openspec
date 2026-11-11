# Tasks: `NarrativeContext.attach`

## 1. arete core: `NarrativeContext` API

- [x] 1.1 Create `NarrativeContext` in `arete` core mirroring `StepContext`: `ThreadLocal<ExtensionContext>`, `static current()` throwing `IllegalStateException` when no narrative scope is active, `ATTACHMENT_NAME_PREFIX = "arete-narrative-attach"`, `attach(name, mediaType, bytes)` publishing via `publishFile` with path separators sanitized to `_`. Verify with a new `NarrativeContextTest` asserting `current()` throws outside a narrative and attaches inside one.
- [x] 1.2 Create `NarrativeContextExtension` implementing `InvocationInterceptor`, holding the class-level `ExtensionContext` during `interceptBeforeAllMethod`, `interceptAfterAllMethod`, and `interceptTestClassConstructor`, clearing in `finally`. Verify with `NarrativeContextExtensionTest` that the context is available in all three and cleared after failure.
- [x] 1.3 Meta-annotate `@Narrative` with `@ExtendWith(NarrativeContextExtension.class)` (attributes unchanged). Verify `./gradlew test` in `arete` passes.

## 2. arete-gradle: reception and plan storage

- [x] 2.1 In `AreteTestListener.fileEntryPublished`, accept the `arete-narrative-attach.` prefix and route to a new `SpecificationPlan.addNarrativeAttachment(uniqueId, name, path)`. Verify with a listener test that a published narrative-attach file lands in the new map, not the step-attachment map.
- [x] 2.2 Add `narrativeAttachmentFiles` map plus `addNarrativeAttachment` / `narrativeAttachments` / `removeNarrativeAttachments` to `SpecificationPlan`; expose `narrativeAttachments` / `hasNarrativeAttachments` on `SpecificationStep`. Verify with a `SpecificationPlan` unit test.

## 3. arete-gradle: copying and rendering

- [x] 3.1 In `HtmlWriter.copyResources`, copy narrative attachments into `specs/` as `<uniqueHash>-<name>`; add `deleteOriginalNarrativeAttachments` and call it from `writeSpec` alongside `deleteOriginalAttachments`. Verify with an `HtmlWriter` test that the copied file exists under the hashed name and the original is deleted.
- [x] 3.2 In `spec.ftlh`, render `step.narrativeAttachments` inside the `step.hasNarrative` block (independent of `narrative.hasAttachments`) with `href="./<uniqueHash>-<name>"`, file icon + name. Verify by rendering a spec with a narrative that has only a dynamic attachment.

## 4. End-to-end verification

- [x] 4.1 Add a demo narrative attachment (e.g. attach in `@AfterAll`) in `arete-gradle-test` and run `./gradlew test` there; verify the generated `specs/*.html` contains the attachment link and the copied file. 
- [x] 4.2 Run `./gradlew test` in `arete` and `arete-gradle`; verify all existing tests still pass (backward compatibility).

## Workflow follow-up

- Archive the change in all affected repos after review requirements are satisfied.
- Verify the archived result.
