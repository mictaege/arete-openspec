# Design: JUnit extension support in `arete-ktx`

## Context

Java arete specs are jupiter test classes, so `@ExtendWith`/`@RegisterExtension` work out of the
box. The Kotlin DSL builds its own descriptor tree executed by `AreteKtxEngine`; jupiter never
sees the spec class as a test container, so its extension model does not apply. We need a
Kotlin-idiomatic registration plus engine-side callback dispatch.

## Decisions

### D1: DSL shape — `extendWith(...)` and `registerExtension(...)` on every container scope

**Decision:** Add to the base scope (available in `spec`, `feature`, `scenario`, `describe`,
`journey`): `extendWith(vararg ext: KClass<out Extension>)` for declarative registration (like
`@ExtendWith`, taking extension classes instantiated via no-arg constructor) and
`registerExtension(ext: Extension)` for programmatic registration (like `@RegisterExtension`,
allowing configured instances). Extensions registered on a container apply to that container's
whole subtree; child registrations add to (not replace) parent ones, mirroring jupiter's
inheritance semantics.

**Rationale:** Matches jupiter semantics users already know (`@ExtendWith` takes classes,
`@RegisterExtension` takes instances); keeps the builder readable (no annotation needed on
lambdas, which Kotlin cannot annotate anyway).

### D2: Engine dispatch with a lightweight `ExtensionContext` facade

**Decision:** `AreteKtxEngine` keeps the registered extensions per `KtxNode`. Around each
container it calls `BeforeAllCallback`/`AfterAllCallback`; around each leaf step,
`BeforeEachCallback`/`AfterEachCallback`; step exceptions go through
`TestExecutionExceptionHandler`; and `TestWatcher` callbacks (`testSuccessful`/`testFailed`/
`testAborted`/`testDisabled`) fire for leaf nodes only (matching jupiter, where `TestWatcher`
is not invoked for containers). A minimal `ExtensionContext` implementation exposes the
current `TestDescriptor`/`KtxNode`, display name, and an `ExtensionContext.Store` backed by a
per-node map (so `RegisterExtension`-style state sharing works). `ExtensionContext.publishFile`
delegates to `KtxEngineBridge` so extensions like `ScreenshotExtension` can publish files.
Extensions needing jupiter's full context (e.g. `ParameterResolver` for `@Test` methods) are
out of scope — the DSL steps take no parameters.

**Rationale:** Reuses jupiter's `Extension` interface unchanged, so existing extensions
(`SpringExtension`, `MockitoExtension`, `ScreenshotExtension`, …) can be registered as-is.

### D3: Ordering and failure semantics follow jupiter

**Decision:** Callbacks run in registration order (before) and reverse order (after); an
exception-handler that swallows the throwable marks the step as aborted like jupiter.

### D4: Legacy `screenshots()` DSL removed

**Decision:** The old `screenshots(taker)` / `screenshots(takeWhen, bytes)` DSL methods and the
engine-internal `ScreenshotConfig`/`publishScreenshot` machinery are removed. Screenshots are now
taken exclusively via `registerExtension(ScreenshotExtension(taker))`.

**Rationale:** The extension mechanism subsumes the bespoke screenshot path; one mechanism is
simpler to maintain and document.

## Affected Components

- `arete-ktx`: `KtxDsl.kt` (registration methods), `KtxNode` (extension list), `AreteKtxEngine`
  (dispatch), new `KtxExtensionContext` facade.
- `arete-gradle-test/kotlin-specs`: all sample specs migrated from `screenshots()` to
  `registerExtension(ScreenshotExtension(...))`.
- No changes to `arete` / `arete-gradle`.
