# Proposal: Kotlin DSL artifact (`arete-ktx`)

Backlog idea #1 from the arete improvement discussion.

## Why

Arete's API is annotation-driven Java (`@Spec`/`@Scenario`/`@Given`…). Kotlin users get no
idiomatic way to write specs: annotations plus nested classes feel unnatural in Kotlin, and there
is no `arete-ktx`-style DSL like other Kotlin testing libraries offer.

## What Changes

- New small Kotlin artifact `arete-ktx` in the arete family, published to Maven Central under
  `io.github.mictaege` like `arete`, depending on `arete` core.
- Idiomatic DSL covering the **full arete functionality**, not just the core styles:
  - **Gherkin style**: `spec` / `feature` / `scenario` / `given` / `` `when` `` / `then`
  - **Descriptive style**: `describe` / `itShould`
  - **Journey style**: `journey` / `step` (with phases), `variableJourney` (with variants)
  - **Examples**: `examples` / `exampleGrid` / `exampleCsv`
  - **Narratives**: `narrative` (text, markdown, images, PlantUML, attachments)
  - **Attachments**: step-level and narrative-level attach
  - **Tags**: tag/stereotype support
  - **References**: `ref` / `seeAlso`
  - Example:
    ```kotlin
    spec("Calculator") {
        scenario("adds 5 to 10") {
            given { a = 5; b = 10 }
            `when` { c = calculator.add(a, b) }
            then { assertThat(c).isEqualTo(15) }
        }
    }
    ```
- The DSL builds the same JUnit extension model underneath, so arete-gradle reports work unchanged.
- **Same JUnit test tree granularity**: every DSL scope (`spec`, `feature`, `scenario`,
  `journey`, `describe`) emits its own selectable `TestDescriptor`. Individual nested elements
  (e.g. a single scenario) can be run from the IDE or via `./gradlew test --tests "..."`,
  exactly as with Java `@Nested` classes.

## Acceptance Criteria

- A Kotlin reference implementation of `travel_agency` lives in `arete-gradle-test` under a
  separate package (e.g. `travel_agency_ktx`) or source set, mirroring the Java
  `travel_agency` specs.
- Running the Kotlin reference implementation produces **exactly the same arete HTML report**
  as the Java reference implementation.

## Out of Scope

- Changes to the Java annotation API of `arete` core (stays stable).
- Report/plugin changes in arete-gradle (none expected — same test model).

## Affected Repos

- new: `arete-ktx` module (family store item)
- `arete` (dependency only)
- `arete-gradle-test` (Kotlin reference implementation in `travel_agency_ktx` package/source set)
