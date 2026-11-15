# Tasks

## 1. Scaffold arete-ktx module

- [ ] 1.1 Create new `arete-ktx` repository with Gradle wrapper, `build.gradle.kts` (Kotlin DSL, `org.jetbrains.kotlin.jvm` plugin, `java-library`, `maven-publish`, `org.jreleaser`), Apache-2.0 LICENSE, and verify `./gradlew build` succeeds
- [ ] 1.2 Add dependency on `io.github.mictaege:arete` (same version) and Kotlin stdlib in `build.gradle.kts`, verify `./gradlew dependencies` resolves correctly
- [ ] 1.3 Configure JUnit Jupiter test dependencies and `useJUnitPlatform()` in the test task, verify `./gradlew test` runs (even with zero tests)

## 2. Core DSL — descriptor tree + Gherkin style

- [ ] 2.1 Implement the dynamic `TestDescriptor` tree infrastructure so each DSL scope emits its own selectable node, verify a minimal `spec("Calculator") { scenario("s") { given {}; `when` {}; then {} } }` produces a tree where the scenario is individually selectable via `./gradlew test --tests "*s*"`
- [ ] 2.2 Implement `spec(name, block)` / `feature(name, block)` / `scenario(name, block)` with correct nesting, verify scenarios appear as nested containers in the JUnit tree
- [ ] 2.3 Implement `given { }`, `` `when` { } ``, `then { }` with Given→When→Then ordering, verify `./gradlew test` shows ordered steps
- [ ] 2.4 Write tests for the Gherkin DSL, verify all pass with `./gradlew test`

## 3. Descriptive style

- [ ] 3.1 Implement `describe(name, block)` (nested) and `itShould(name, block)`, verify expectations appear as individually selectable tests via `./gradlew test --tests "*itShould*"`
- [ ] 3.2 Write tests for the descriptive DSL, verify all pass with `./gradlew test`

## 4. Journey + VariableJourney style

- [ ] 4.1 Implement `journey(name, block)` and `step(order, phase?) { }` with ordered execution, verify individual steps are selectable and run in declared order
- [ ] 4.2 Implement `variableJourney(name, block)` with variant support (`step(order, variant = "...")`), verify each variant runs as a separate selectable journey
- [ ] 4.3 Write tests for journey and variable journey DSL, verify all pass with `./gradlew test`

## 5. Examples / ExampleGrid / ExampleCsv

- [ ] 5.1 Implement `examples(pattern, test, source)` DSL mapping to JUnit `@TestTemplate`, verify parameterized examples run and appear correctly in the report
- [ ] 5.2 Implement `exampleGrid { }` and `exampleCsv { }` DSL, verify tabular examples run
- [ ] 5.3 Write tests for all example DSL forms, verify all pass with `./gradlew test`

## 6. Narrative, attachments, tags, refs, seeAlso

- [ ] 6.1 Implement `narrative(...)` DSL (text/markdown/images/PlantUML/attachments), verify narrative appears in the arete report
- [ ] 6.2 Implement step-level and narrative-level `attach(...)` DSL, verify attachments show in the report
- [ ] 6.3 Implement tag/stereotype DSL, verify tags appear in the report tag page
- [ ] 6.4 Implement `ref(value, url)` and `seeAlso(KClass)` DSL, verify references/links appear in the report
- [ ] 6.5 Write tests for all of the above, verify all pass with `./gradlew test`

## 7. Kotlin reference implementation (acceptance)

- [ ] 7.1 Add `arete-ktx` as a test dependency in `arete-gradle-test` (via `includeBuild("../arete-ktx")` in `settings.gradle.kts`)
- [ ] 7.2 Create `travel_agency_ktx` package (or source set) in `arete-gradle-test` mirroring all 6 Java `travel_agency` specs using the Kotlin DSL
- [ ] 7.3 Run `./gradlew test` in `arete-gradle-test` and verify the Kotlin `travel_agency_ktx` specs produce **exactly the same arete HTML report** as the Java `travel_agency` specs

## 8. Documentation and publishing setup

- [ ] 8.1 Write `Readme.md` for `arete-ktx` with DSL examples covering all features, verify examples compile as documented
- [ ] 8.2 Configure jreleaser publishing in `build.gradle.kts` for Maven Central under `io.github.mictaege:arete-ktx`, verify `./gradlew publishToMavenLocal` succeeds
