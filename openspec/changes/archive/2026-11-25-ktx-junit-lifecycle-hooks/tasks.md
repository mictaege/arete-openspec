# Tasks

## 1. DSL surface in arete-ktx

- [x] 1.1 Add hook fields to `KtxNode` (`beforeAllHook`, `beforeEachHook`, `afterEachHook`, `afterAllHook`) and subsume `preBody` into `beforeAllHook` in `KtxDsl.kt`; verify by compiling arete-ktx with `./gradlew compileKotlin`.
- [x] 1.2 Add `beforeAll`/`beforeEach`/`afterEach`/`afterAll` methods with context-scope receiver to `KtxScope` (single-slot semantics), and `afterVariant` plus context-scope `beforeVariant` to `VariableJourneyScope`; verify by compiling and by a scratch spec using all hooks.
- [x] 1.3 Add unit tests in arete-ktx covering DSL registration: hook stored on the right node, single-slot replacement, variant nodes get before/afterVariant; verify with `./gradlew test --tests '*Lifecycle*'`.

## 2. Engine dispatch in arete-ktx

- [x] 2.1 Invoke `beforeAllHook` once per container before children and `afterAllHook` in `finally` after children in `AreteKtxEngine.execute`; verify with engine-level tests asserting invocation counts.
- [x] 2.2 Invoke `beforeEachHook`/`afterEachHook` around leaf bodies with parent-first/reverse accumulation and `finally` semantics; establish `KtxStepContext`/`KtxNarrativeContext` so `attachToStep`/`attachToNarrative` work in hooks; verify with tests covering success, failure, and attachment from a hook.
- [x] 2.3 Wire `afterVariant` onto variant nodes in `SpecScope.variableJourney`; verify with a variable-journey test asserting per-variant before/after ordering.

## 3. End-to-end validation

- [x] 3.1 Add travel_agency Kotlin spec coverage in arete-gradle-test exercising all six hooks; verify with `./gradlew test` in arete-gradle-test and the arete-gradle report showing attached files.
- [x] 3.2 Run full `./gradlew test` in arete-ktx and arete to confirm no regressions; verify all tests green.
