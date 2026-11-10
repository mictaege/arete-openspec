# Proposal: `StepContext.attach` — attach arbitrary files to the current step

Backlog idea #7b from the arete improvement discussion (split into two changes). Proposal-only.

## Why

Today only `ScreenshotTaker` can attach files to a test (as `arete-screenshot.*` JUnit file
attachments). Any step code — HTML dumps, API responses, logs, HAR files — should be able to attach
files to the *current step* so they show up in the arete report next to that step.

## What Changes

- arete core: public `StepContext` API (reachable during step execution, e.g. via
  `ExtensionContext` store or a static current-step holder) with
  `attach(name, mediaType, bytes)`; publishes via JUnit `publishFile` under a namespace prefix
  (e.g. `arete-attach.<name>`), distinct from `arete-screenshot.`.
- arete-gradle: generalize `AreteTestListener.fileEntryPublished` to accept the `arete-attach.`
  prefix in addition to screenshots; register attachments in `SpecificationPlan` by uniqueId;
  `HtmlWriter.copyResources` copies them into `specs/`; `spec.ftlh` renders download links with a
  file icon on the owning step.
- Delta specs: new capability spec `attachments` in arete-gradle; `screenshots` spec in arete
  extended or a new `step-context` spec.

## Out of Scope

- Screenshot behavior (already works, stays on `arete-screenshot.`).
- Storing attachments in the history store (possible follow-up).
- Size limits/cleanup beyond the existing screenshot cleanup rules.

## Affected Repos

- `arete` (public `StepContext` API + publishing)
- `arete-gradle` (reception, copying, rendering)
