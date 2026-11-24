# Spec Delta

## Purpose

Kotlin DSL specs can register JUnit Jupiter `Extension`s (as with Java `@ExtendWith` /
`@RegisterExtension`) and the `arete-ktx` engine invokes their lifecycle callbacks around the
corresponding spec tree nodes.

## ADDED Requirements

### Requirement: Extensions registrable from the DSL

Every container scope (`spec`, `feature`, `scenario`, `describe`, `journey`) SHALL offer
`extendWith(vararg KClass<out Extension>)` for declarative registration (like Java's
`@ExtendWith`, instantiating via no-arg constructor) and `registerExtension(Extension)` for
programmatic registration (like Java's `@RegisterExtension`, allowing configured instances).

#### Scenario: Register extension class in a spec

- **WHEN** a Kotlin spec calls `extendWith(MyExtension::class)` in the `spec { ... }` block
- **THEN** the extension is instantiated and its callbacks wrap the execution of that spec's subtree

#### Scenario: Register extension instance in a spec

- **WHEN** a Kotlin spec calls `registerExtension(MyExtension(config))` in the `spec { ... }` block
- **THEN** the configured instance's callbacks wrap the execution of that spec's subtree

### Requirement: Lifecycle callbacks are invoked around nodes

The engine SHALL invoke `BeforeAllCallback`/`AfterAllCallback` around container nodes,
`BeforeEachCallback`/`AfterEachCallback` around leaf step nodes, `TestWatcher` callbacks
(`testSuccessful`/`testFailed`/`testAborted`/`testDisabled`) for leaf nodes only, and SHALL
route step failures through `TestExecutionExceptionHandler` when registered.

#### Scenario: Step failure handled by extension

- **WHEN** a step body throws and an extension implements `TestExecutionExceptionHandler`
- **THEN** the handler is invoked with the exception before the failure is recorded

#### Scenario: TestWatcher fires for steps only

- **WHEN** an extension implements `TestWatcher` and is registered on a container
- **THEN** `testSuccessful`/`testFailed`/`testAborted` fire only for leaf step nodes, not for containers

### Requirement: ExtensionContext supports file publishing

The engine's `ExtensionContext` facade SHALL support `publishFile(...)` so extensions that
publish attachments (e.g. `ScreenshotExtension`) work unchanged.

#### Scenario: ScreenshotExtension publishes screenshots

- **WHEN** `registerExtension(ScreenshotExtension(taker))` is used on a spec
- **THEN** screenshots are published as file entries on the corresponding leaf step nodes

### Requirement: No extensions means unchanged behavior

Specs that register no extensions SHALL behave exactly as before (same descriptor tree, same
report output).

#### Scenario: Plain spec unaffected

- **WHEN** a spec uses no extension DSL
- **THEN** execution and the arete report are identical to current behavior
