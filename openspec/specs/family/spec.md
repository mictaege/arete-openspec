# Arete Family Specification

## Purpose

The arete family is four GitHub repositories under `github.com/mictaege` that are developed and
released together: the `arete` core library, the `arete-ktx` Kotlin DSL artifact, the
`arete-gradle` reporting plugin, and the `arete-gradle-test` consumer/demo project. This store
(`openspec store arete`, at `~/openspec/arete`) holds family-level planning; each repo keeps its
own `openspec/` root for repo-local specs and changes.

## Requirements

### Requirement: Three repos with a one-way dependency direction

The family SHALL consist of exactly `arete` (core JUnit 6 BDD extension library), `arete-ktx`
(Kotlin DSL artifact depending on `arete`), `arete-gradle` (Gradle HTML report plugin depending on
`arete` and reading `arete-ktx` metadata), and `arete-gradle-test` (sample/integration consumer,
not published). Dependencies SHALL only point `arete-ktx` → `arete`, `arete-gradle` → `arete` +
`arete-ktx`, and `arete-gradle-test` → all; `arete` SHALL NOT depend on the others.

#### Scenario: Consumer validates plugin changes locally
- **WHEN** a change is made in `arete` or `arete-gradle`
- **THEN** `arete-gradle-test` picks it up via `includeBuild("../arete")` /
  `includeBuild("../arete-gradle")` in its `settings.gradle.kts`, so `./gradlew test` there
  exercises the local code without publishing

### Requirement: Java/Kotlin parity for features

Any new feature or functionality in the Java annotation API of `arete` SHALL be checked for a
Kotlin DSL equivalent in `arete-ktx`, and vice versa: a new Java annotation (e.g. a new
`@Spec`-level annotation) SHALL be evaluated for a corresponding DSL element in `arete-ktx`, and
a new Kotlin DSL element SHALL be evaluated for a Java annotation equivalent. The parity decision
(implement / consciously skip with rationale) SHALL be recorded in the OpenSpec change of the
affected repos.

#### Scenario: New Java annotation is added
- **WHEN** a new annotation is added to `arete` (e.g. a new metadata annotation)
- **THEN** the change records whether `arete-ktx` needs a matching DSL element, and if so it is
  implemented in the same release

#### Scenario: New Kotlin DSL element is added
- **WHEN** a new builder/element is added to the `arete-ktx` DSL
- **THEN** the change records whether the Java API needs a matching annotation, and if so it is
  added to `arete` in the same release

### Requirement: Shared versioning and release flow

All four repos SHALL carry the same version per release (e.g. `2026.8`). Releases SHALL flow via
a `release/YYYY.N` branch merged into `master` and tagged; `develop` is the integration branch.

#### Scenario: Cross-repo feature lands in one release
- **WHEN** a feature touches `arete` and `arete-gradle` (e.g. `@Ref` support)
- **THEN** each repo gets a same-named OpenSpec change, both are implemented on `develop`, and both
  are archived and released together in the same version

### Requirement: Publishing targets

`arete` SHALL be published to Maven Central as `io.github.mictaege:arete`; `arete-gradle` to Maven
Central as `io.github.mictaege:arete-gradle` and to the Gradle Plugin Portal as
`io.github.mictaege.arete`; `arete-gradle-test` SHALL NOT be published. All use org.jreleaser
(Maven Central) / com.gradle.plugin-publish (portal), with credentials from Gradle
properties/environment only.

#### Scenario: Plugin applied by id
- **WHEN** a consumer applies `id("io.github.mictaege.arete") version "2026.8"`
- **THEN** the plugin resolves from the Gradle Plugin Portal and pulls `io.github.mictaege:arete`
  of the same version from Maven Central

### Requirement: API stability of the core library

The public annotation/extension API of `arete` SHALL stay backward compatible across releases,
because `arete-gradle` (and external consumers) read it at runtime to build reports; breaking
changes SHALL be called out in release notes.

#### Scenario: Report keeps working across a minor release
- **WHEN** `arete-gradle` is built against a newer `arete` with an unchanged public API
- **THEN** report generation works unchanged; any behavioral change in the report model is
  proposed as an OpenSpec change in both repos first
