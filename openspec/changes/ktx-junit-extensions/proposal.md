# Proposal: JUnit extension support in the Kotlin DSL (`arete-ktx`)

## Why

Java arete specs can combine any JUnit Jupiter feature — `@ExtendWith(...)`,
`@RegisterExtension`, `@BeforeEach`/`@AfterEach`, parameter resolvers, execution-condition
and exception-handler extensions — because the spec is a real class and jupiter's extension
model applies. The Kotlin DSL (`Spec({ ... })`) has no equivalent: there is no class or method
to annotate and no extension registry, so Kotlin specs cannot reuse jupiter extensions
(e.g. `SpringExtension`, `MockitoExtension`, arete's own `ScreenshotExtension`-style
callbacks, `@Timeout`-like wrappers).

## What Changes

- New DSL element(s) in `arete-ktx` available in every container scope (`spec`, `feature`,
  `scenario`, `describe`, `journey`), e.g. `extensions(vararg ext: Extension)` /
  `registerExtension(ext: Extension)` mirroring `@ExtendWith` / `@RegisterExtension`.
- The `AreteKtxEngine` invokes the registered extensions' lifecycle callbacks around the
  corresponding node execution (before/after container, before/after step, exception
  handling), using a minimal `ExtensionContext`-like facade backed by the `KtxNode`/
  `TestDescriptor`.
- Existing behavior is unchanged when no extensions are registered.

## Capabilities

### New Capabilities

- `ktx-extensions`: register JUnit Jupiter `Extension`s from the Kotlin DSL and have the
  `arete-ktx` engine invoke their lifecycle callbacks around spec tree nodes.

### Modified Capabilities

- (none)

## Impact

- `arete-ktx`: new DSL surface + engine callback dispatch. Additive only.
- `arete-gradle`: none (extensions are runtime concerns; report unchanged unless an extension
  publishes attachments, which already works via the existing file-entry channel).
- `arete-gradle-test/kotlin-specs`: add a sample spec combining a jupiter extension.
