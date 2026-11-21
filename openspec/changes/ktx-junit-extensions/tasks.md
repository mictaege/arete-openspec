# Tasks: JUnit extension support in `arete-ktx`

- [ ] 1. Add `extensions(vararg Extension)` / `registerExtension(Extension)` to the base `KtxScope`;
        store extensions on `KtxNode`, inheriting parent registrations down the tree
- [ ] 2. Implement `KtxExtensionContext` facade (descriptor/node, display name, store) in arete-ktx
- [ ] 3. Dispatch `BeforeAllCallback`/`AfterAllCallback` around containers and
        `BeforeEachCallback`/`AfterEachCallback` around steps in `AreteKtxEngine`, registration
        order / reverse order
- [ ] 4. Route step throwables through `TestExecutionExceptionHandler`
- [ ] 5. Add a sample `travel_agency_ktx` spec registering a jupiter extension; verify callbacks
        fire and the arete report is unchanged for specs without extensions
- [ ] 6. Update `arete-ktx/Readme.md` with an extension example
- [ ] 7. Run full family verification (`arete`, `arete-ktx`, `arete-gradle`, `:kotlin-specs:test`,
        `:java-specs:test`)
