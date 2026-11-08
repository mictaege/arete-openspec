# Proposal: Kotlin DSL artifact (`arete-ktx`)

Backlog idea #1 from the arete improvement discussion. Proposal-only — no design/tasks yet.

## Why

Arete's API is annotation-driven Java (`@Spec`/`@Scenario`/`@Given`…). Kotlin users get no
idiomatic way to write specs: annotations plus nested classes feel unnatural in Kotlin, and there
is no `arete-ktx`-style DSL like other Kotlin testing libraries offer.

## What Changes

- New small Kotlin artifact `arete-ktx` in the arete family, published to Maven Central under
  `io.github.mictaege` like `arete`, depending on `arete` core.
- Idiomatic DSL covering all three authoring styles, e.g.:
  ```kotlin
  spec("Calculator") {
      scenario("adds 5 to 10") {
          given { a = 5; b = 10 }
          `when` { c = calculator.add(a, b) }
          then { assertThat(c).isEqualTo(15) }
      }
  }
  ```
  plus descriptive (`describe`/`itShould`) and journey (`journey`/`step`) forms.
- The DSL builds the same JUnit extension model underneath, so arete-gradle reports work unchanged.

## Out of Scope

- Changes to the Java annotation API of `arete` core (stays stable).
- Report/plugin changes in arete-gradle (none expected — same test model).
- Examples/grid DSL sugar (can follow later).

## Affected Repos

- new: `arete-ktx` module (family store item)
- `arete` (dependency only), `arete-gradle-test` (example consumer, optional)
