# Spec Delta

## Purpose

Kotlin DSL specs can register JUnit Jupiter `Extension`s (as with Java `@ExtendWith` /
`@RegisterExtension`) and the `arete-ktx` engine invokes their lifecycle callbacks around the
corresponding spec tree nodes.

## ADDED Requirements

### Requirement: Extensions registrable from the DSL

Every container scope (`spec`, `feature`, `scenario`, `describe`, `journey`) SHALL offer a way
to register `org.junit.jupiter.api.extension.Extension` instances, equivalent to Java's
`@ExtendWith` (declarative) and `@RegisterExtension` (programmatic, e.g. with parameters).

#### Scenario: Register extension in a spec

- **WHEN** a Kotlin spec registers an extension in the `spec { ... }` block
- **THEN** the extension's callbacks wrap the execution of that spec's subtree

### Requirement: Lifecycle callbacks are invoked around nodes

The engine SHALL invoke `BeforeAllCallback`/`AfterAllCallback` around container nodes and
`BeforeEachCallback`/`AfterEachCallback` around leaf step nodes, and SHALL route step failures
through `TestExecutionExceptionHandler` when registered.

#### Scenario: Step failure handled by extension

- **WHEN** a step body throws and an extension implements `TestExecutionExceptionHandler`
- **THEN** the handler is invoked with the exception before the failure is recorded

### Requirement: No extensions means unchanged behavior

Specs that register no extensions SHALL behave exactly as before (same descriptor tree, same
report output).

#### Scenario: Plain spec unaffected

- **WHEN** a spec uses no extension DSL
- **THEN** execution and the arete report are identical to current behavior
