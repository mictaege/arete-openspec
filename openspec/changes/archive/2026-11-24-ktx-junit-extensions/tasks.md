# Tasks: JUnit extension support in `arete-ktx`

- [x] 1. Add `extendWith(vararg KClass<out Extension>)` / `registerExtension(Extension)` to the base `KtxScope`;
        store extensions on `KtxNode`, inheriting parent registrations down the tree
- [x] 2. Implement `KtxExtensionContext` facade (descriptor/node, display name, store, publishFile) in arete-ktx
- [x] 3. Dispatch `BeforeAllCallback`/`AfterAllCallback` around containers and
        `BeforeEachCallback`/`AfterEachCallback` around steps in `AreteKtxEngine`, registration
        order / reverse order
- [x] 4. Route step throwables through `TestExecutionExceptionHandler`
- [x] 5. Dispatch `TestWatcher` callbacks for leaf nodes only (testSuccessful/testFailed/testAborted/testDisabled)
- [x] 6. Remove legacy `screenshots()` DSL and engine-internal screenshot machinery; migrate all
        sample specs to `registerExtension(ScreenshotExtension(taker))`
- [x] 7. Update `arete-ktx/Readme.md` with an extension example
- [x] 8. Run full family verification (`arete`, `arete-ktx`, `arete-gradle`, `:kotlin-specs:test`,
        `:java-specs:test`)
