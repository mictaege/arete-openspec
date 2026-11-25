# Proposal: JUnit lifecycle hooks in the Kotlin DSL (`arete-ktx`)

## Why

Java arete specs get JUnit Jupiter's `@BeforeAll`/`@BeforeEach`/`@AfterEach`/`@AfterAll` for free
because the spec is a real class that JUnit annotates and invokes. The Kotlin DSL (`Spec({ ... })`)
has no equivalent: the only lifecycle hook today is `beforeVariant` on variable journeys, and there
is not even an `afterVariant` counterpart to Java's `@AfterVariant`. Kotlin specs therefore cannot
run setup/teardown around containers or individual steps without registering a hand-written
extension via `registerExtension(...)`.

## What Changes

- New DSL methods on every container scope (`spec`, `feature`, `scenario`, `describe`, `journey`,
  `variableJourney`), mirroring JUnit's lifecycle annotations:
  - `beforeAll { }` — once per container, before its children (and before all variants of a
    variable journey)
  - `beforeEach { }` — around every executable leaf (given/when/then steps, example rows, itShould)
  - `afterEach { }` — around every leaf, executed even when the leaf fails
  - `afterAll { }` — once per container, after its children, executed even on failure
- New `afterVariant { }` on `VariableJourneyScope`, the missing Kotlin counterpart of Java's
  `@AfterVariant`; per-variant teardown running after all steps of that variant.
- `beforeVariant { }` is kept and gains the same context scope as the new hooks.
- All hook lambdas receive a context scope (like `StepBodyScope`) so they can `attachToStep` /
  `attachToNarrative`; `beforeVariant` changes from `() -> Unit` to the context-scope lambda.
- Each hook is a single slot per scope: a second call replaces the first (same semantics as
  `beforeVariant` today).
- The `AreteKtxEngine` invokes the hooks around the corresponding node execution, with
  parent-before-child ordering for `before*` and reverse ordering for `after*`, and establishes
  the narrative/step context so attachments work inside hooks.

## Capabilities

### New Capabilities

- `ktx-lifecycle`: JUnit-style lifecycle hooks (`beforeAll`/`beforeEach`/`afterEach`/`afterAll`,
  plus `beforeVariant`/`afterVariant` for variable journeys) in the arete Kotlin DSL, invoked by
  the arete-ktx engine around spec tree nodes.

### Modified Capabilities

- (none)

## Impact

- `arete-ktx`: new DSL surface in `KtxDsl.kt` (`KtxScope`, `VariableJourneyScope`, `KtxNode`) and
  hook dispatch in `AreteKtxEngine.kt`. Additive only; existing specs without hooks are unchanged.
- `arete`: none (Java side already has `@BeforeVariant`/`@AfterVariant`; the JUnit annotations
  cover the rest). Parity decision: Kotlin DSL methods are the parity counterpart of JUnit's own
  `@BeforeEach`/`@BeforeEach` etc. on Java spec classes, not a new Java annotation.
- `arete-gradle`: none (reporting unchanged; hooks are runtime concerns).
- `arete-gradle-test`: add demo coverage in the travel_agency Kotlin specs exercising the hooks.
