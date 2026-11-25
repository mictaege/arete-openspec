# Design

## Context

The Kotlin DSL (`arete-ktx`) builds a `KtxNode` tree at definition time; `AreteKtxEngine` walks it at
execution time and already dispatches JUnit extension callbacks (`BeforeAllCallback`,
`BeforeEachCallback`, `AfterEachCallback`, `AfterAllCallback`) with inheritance down the tree. The
only DSL lifecycle hook today is `beforeVariant`, stored as `KtxNode.preBody` and invoked once per
node before its children. See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- DSL methods `beforeAll`/`beforeEach`/`afterEach`/`afterAll` on every container scope, plus
  `afterVariant` on variable journeys, all with a context scope for attachments.
- Single-slot semantics per scope; JUnit-style parent-first/reverse ordering; teardown hooks run
  on failure.

**Non-Goals:**
- No changes to the Java `arete` annotation API or to `arete-gradle` reporting.
- No multi-registration stacking of hooks within one scope.
- No parameter injection into hook lambdas beyond the attachment context scope.

## Decisions

**D1 — Storage on KtxNode.** Each hook is a single-slot lambda field on `KtxNode`
(`beforeAllHook`, `beforeEachHook`, `afterEachHook`, `afterAllHook`), replacing the special-purpose
`preBody`. `beforeAll` and the existing `preBody` are the same semantic, so `preBody` is subsumed:
`beforeVariant` sets the variant node's `beforeAllHook`. Single slot matches the decided semantics
and keeps `KtxNode` simple.

**D2 — Hook lambda type.** Hooks take a context scope: `LifecycleScope.() -> Unit`, where
`LifecycleScope` exposes `attachToStep`/`attachToNarrative` (same surface as `StepBodyScope`,
possibly by making `StepBodyScope` itself the receiver). This changes `beforeVariant` from
`() -> Unit` to the context-scope lambda — source-compatible for existing callers that ignore the
receiver.

**D3 — Engine dispatch with inheritance.** `execute()` already threads `inheritedExtensions`; the
hook lists are accumulated the same way: effective beforeEach = parent chain + own, invoked
parent-first; afterEach/afterAll invoked child-first (reversed). `afterEach`/`afterAll` run in
`finally` blocks so they execute on failure, mirroring the extension callback handling.

**D4 — Context establishment.** `KtxNarrativeContext` is already swapped per container before
children execute; hooks at container level run inside that scope so `attachToNarrative` targets the
right node. `KtxStepContext` is set around leaf bodies; `beforeEach`/`afterEach` run inside that
same bracket so `attachToStep` works.

**D5 — variableJourney semantics.** before/afterAll on a `VariableJourneyScope` attach to the
variableJourney node itself (once around all variants); before/afterVariant attach to each variant
node (per variant).

## Risks / Trade-offs

- [Risk] Changing `beforeVariant`'s lambda type breaks binary compatibility for compiled consumers
  → Mitigation: it is a source-level DSL; lambda with receiver is source-compatible for existing
  call sites; note in release notes.
- [Risk] Hooks running in `finally` can mask the original failure if they throw → Mitigation:
  wrap hook invocation in `runCatching` and add suppressed exceptions, same style as the existing
  `TestWatcher` dispatch.

## Migration Plan

None required — purely additive DSL surface; existing specs behave identically when no hooks are
declared.

## Open Questions

- None blocking; exact naming of the context-scope receiver (reuse `StepBodyScope` vs. new
  `LifecycleScope`) can be settled during implementation.
