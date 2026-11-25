# Spec Delta

## Purpose

Gives the arete Kotlin DSL JUnit-style lifecycle hooks (`beforeAll`/`beforeEach`/`afterEach`/`afterAll`)
plus per-variant `beforeVariant`/`afterVariant`, so Kotlin specs can run setup and teardown around
containers and individual steps without registering hand-written extensions.

## ADDED Requirements

### Requirement: beforeAll hook on container scopes
Every container scope (spec, feature, scenario, describe, journey, variableJourney) SHALL provide
`beforeAll { }`, invoked exactly once per container execution before any of its children run. For a
variable journey, the variableJourney-level `beforeAll` SHALL run once before all variants.

#### Scenario: beforeAll runs once per container
- **WHEN** a spec defines `beforeAll` in a feature and the feature contains two scenarios
- **THEN** the hook body executes exactly once, before either scenario's steps

#### Scenario: variableJourney beforeAll wraps all variants
- **WHEN** a variable journey defines `beforeAll` and steps in two variants
- **THEN** the hook body executes exactly once, before the first step of the first variant

### Requirement: beforeEach/afterEach hooks around leaf steps
Every container scope SHALL provide `beforeEach { }` and `afterEach { }`, invoked around every
executable leaf in its subtree (given/when/then steps, example rows, itShould). `afterEach` SHALL
run even when the leaf fails.

#### Scenario: beforeEach/afterEach wrap each step
- **WHEN** a scenario defines `beforeEach` and `afterEach` and contains a given and a then step
- **THEN** beforeEach runs before each step and afterEach runs after each step (four invocations total)

#### Scenario: afterEach runs on failure
- **WHEN** a step throws an assertion error and the enclosing scope defines `afterEach`
- **THEN** afterEach still executes before the failure is reported

### Requirement: afterAll hook on container scopes
Every container scope SHALL provide `afterAll { }`, invoked exactly once per container execution
after all its children have run, even when a child failed. For a variable journey, the
variableJourney-level `afterAll` SHALL run once after all variants.

#### Scenario: afterAll runs once after children
- **WHEN** a feature defines `afterAll` and contains two scenarios
- **THEN** the hook body executes exactly once, after both scenarios complete

### Requirement: hooks inherit parent-first, teardown reverse
Hooks SHALL compose down the tree: parent `beforeAll`/`beforeEach` run before child hooks, and
`afterEach`/`afterAll` run in reverse (child before parent), matching JUnit's ordering.

#### Scenario: nested scopes compose hooks
- **WHEN** a spec defines `beforeEach` and a nested scenario defines its own `beforeEach`
- **THEN** the spec-level hook runs first, then the scenario-level hook, before each leaf step

### Requirement: afterVariant hook on variable journeys
`VariableJourneyScope` SHALL provide `afterVariant { }`, invoked once per variant after all steps of
that variant have run, mirroring Java's `@AfterVariant`.

#### Scenario: afterVariant runs per variant
- **WHEN** a variable journey defines `afterVariant` and steps in two variants
- **THEN** the hook body executes once after each variant's steps (two invocations total)

### Requirement: hooks receive a context scope for attachments
All hook lambdas (`beforeAll`, `beforeEach`, `afterEach`, `afterAll`, `beforeVariant`,
`afterVariant`) SHALL receive a context scope exposing `attachToStep` and `attachToNarrative`, so
files can be attached from inside hooks.

#### Scenario: attachment from a hook
- **WHEN** a `beforeEach` hook calls `attachToStep` with a file
- **THEN** the file is attached to the step that follows the hook

### Requirement: single-slot semantics per scope
Each hook SHALL be a single slot per scope: calling the same hook twice in one scope SHALL replace
the previous registration rather than stack.

#### Scenario: repeated registration replaces
- **WHEN** a scope calls `beforeEach` twice
- **THEN** only the second hook body executes around each leaf
