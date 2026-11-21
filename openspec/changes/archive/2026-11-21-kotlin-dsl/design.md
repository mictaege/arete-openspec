# Design

## Context

Arete core (`arete`) is a Java annotation-driven JUnit 6 extension. Its public API consists of
annotations (`@Spec`, `@Scenario`, `@Given`/`@When`/`@Then`, `@Describe`/`@ItShould`,
`@Journey`/`@Step`, `@VariableJourney`, `@Feature`, `@Narrative`, `@Ref`, `@SeeAlso`,
`@Examples`, `@ExampleGrid`, `@ExampleCsv`, etc.) meta-annotated with JUnit Jupiter annotations
(`@Nested`, `@TestInstance`, `@TestMethodOrder`, `@DisplayNameGeneration`, ...). Runtime
behavior is driven by `AreteOrderer` and the name generators. See proposal.md for motivation.

The Kotlin DSL (`arete-ktx`) must produce the same JUnit extension model so that `arete-gradle`
reports work unchanged — including the exact same report output as the Java reference
implementation.

## Goals / Non-Goals

**Goals:**
- Idiomatic Kotlin DSL covering the **full arete feature set**: Gherkin, descriptive, journey,
  variable journey, examples/grid/csv, narratives, attachments, tags, refs, seeAlso.
- Same JUnit test tree granularity as Java: every DSL scope emits its own selectable
  `TestDescriptor` so individual nested elements can be run from IDE or Gradle `--tests`.
- Same arete HTML report output as the Java reference implementation.
- New standalone artifact `arete-ktx` depending only on `arete` core + Kotlin stdlib.
- Published to Maven Central as `io.github.mictaege:arete-ktx`.

**Non-Goals:**
- No changes to the Java annotation API of `arete` core.
- No report/plugin changes in `arete-gradle`.
- No new repo for the reference implementation — it lives in `arete-gradle-test`.

## Decisions

### D1: Dynamic JUnit test registration with a proper descriptor tree

**Decision:** The DSL builds on JUnit Jupiter's dynamic test registration, emitting a
`TestDescriptor` tree that mirrors the Java `@Nested` class hierarchy — not a flat list.

**Rationale:** Requirement 2 demands that individual scenarios/journeys/features be selectable
and runnable from IDE and Gradle `--tests` filters, exactly as with Java `@Nested` classes.
Each DSL scope function (`spec`, `feature`, `scenario`, `journey`, `describe`) registers its
own container descriptor with child descriptors underneath.

**Known limitation (documented, not fulfilled):** the descriptor tree gives the IDE run tree and
the arete report the same granularity as Java, but the scopes are not *selectable* for execution.
Unlike Java `@Nested` classes (real classes, hence addressable by Gradle `--tests` and IDE run
markers), all Kotlin DSL scopes live in one class constructor lambda, so Gradle `--tests` filters
at the spec-class level only and the IDE can only run a whole spec class (the `@Test` marker on
`Spec` provides the class-level gutter icon). Individual scenarios/features/journeys cannot be
executed on their own. See `arete-ktx/Readme.md` "Known Limitations".

**Alternatives considered:**
- *Single `@TestFactory` per spec* — rejected: produces a flat test list, breaking individual
  selection.
- *Kotlin compiler plugin generating annotated classes* — rejected: too much complexity.

### D2: DSL scope functions map 1:1 to existing arete concepts

**Decision:** Each DSL function maps to an existing arete annotation concept:

| DSL | Maps to |
|-----|---------|
| `spec("Name") { ... }` | `@Spec` class |
| `feature("Name") { ... }` | `@Feature` nested class |
| `scenario("Name") { ... }` | `@Scenario` nested class |
| `given { }` / `` `when` { } `` / `then { }` | `@Given` / `@When` / `@Then` methods |
| `describe("Name") { ... }` | `@Describe` nested class |
| `itShould("Name") { ... }` | `@ItShould` method |
| `journey("Name") { ... }` | `@Journey` nested class |
| `step(order) { ... }` | `@Step(order)` method |
| `variableJourney("Name") { ... }` | `@VariableJourney` nested class |
| `examples(pattern) { ... }` | `@Examples` |
| `exampleGrid { ... }` / `exampleCsv { ... }` | `@ExampleGrid` / `@ExampleCsv` |
| `narrative("...") { ... }` | `@Narrative` |
| `ref("PROJ-1", url = "...")` | `@Ref` |
| `seeAlso(OtherSpec::class)` | `@SeeAlso` |

**Rationale:** 1:1 mapping guarantees the same report output and a consistent mental model
between Java and Kotlin users.

### D3: Module structure — new standalone Gradle module `arete-ktx`

**Decision:** `arete-ktx` is a separate repo/module (like `arete` and `arete-gradle`), not a
subproject of `arete`. It depends on `arete` core. The reference implementation lives in
`arete-gradle-test` under package `travel_agency_ktx` (or a separate source set).

**Rationale:** Keeps `arete` core pure Java and lightweight. Follows the family pattern of
small focused leaf modules. Published under `io.github.mictaege`.

### D4: State sharing within scenarios/journeys via DSL receiver

**Decision:** The scenario/journey block receiver exposes mutable state so steps can share it,
matching arete's PER_CLASS lifecycle:

```kotlin
scenario("adds 5 to 10") {
    val a = 5
    val b = 10
    `when` { result = calculator.add(a, b) }
    then { assertThat(result).isEqualTo(15) }
}
```

### D5: Examples/grid/csv via lambda-based source blocks

**Decision:** `@Examples`/`@ExampleGrid`/`@ExampleCsv` are expressed as DSL blocks taking a
source lambda, mapped to JUnit `@TestTemplate` invocation contexts:

```kotlin
scenario("adds two numbers") {
    examples("{0} + {1} => {2}") { (a, b, expected) ->
        assertThat(calculator.add(a, b)).isEqualTo(expected)
    } source {
        example(given(3), given(4), then(7))
        example(given(-3), given(4), then(1))
    }
}
```

**Rationale:** Kotlin lambdas replace the Java `ExampleSource` method/class mechanism while
producing the same parameterized test model.

## Risks / Trade-offs

- [Risk: Dynamically registered tests may produce different display names than annotation-based ones] → Mitigation: reuse the same name generators (`ScenarioNameGenerator`, etc.) when building descriptors.
- [Risk: arete-gradle report may not fully recognize dynamically registered tests] → Mitigation: acceptance test — Kotlin `travel_agency_ktx` must produce a byte-identical report to the Java `travel_agency`.
- [Risk: Full feature coverage makes v1 large] → Mitigation: implement in order (core styles → journey/variable → examples → narrative/refs), each verified against the Java reference.

## Migration Plan

No migration needed — new artifact. Consumers add `testImplementation("io.github.mictaege:arete-ktx:YYYY.N")`.

## Open Questions

- Should `travel_agency_ktx` live in a separate Gradle source set or just a different package in the existing test source set? (Either is acceptable per proposal; decide during implementation.)
