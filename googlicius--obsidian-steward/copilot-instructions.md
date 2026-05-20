## test-rule

> - When setting up a new test that requires mocking the plugin, create the `createMockPlugin` function within that test file with the necessary mocked properties. Finds an example here: @src/services/UserDefinedCommandService.test.ts (0-40)

- When setting up a new test that requires mocking the plugin, create the `createMockPlugin` function within that test file with the necessary mocked properties. Finds an example here: @src/services/UserDefinedCommandService.test.ts (0-40)
- When setting up a new test, write only 2 or 3 test cases to ensure it work before writing other cases.
- Set up a mock that bypasses Typescript: @src/solutions/commands/agents/handlers/VaultList.test.ts:32-36

---
> Source: [googlicius/obsidian-steward](https://github.com/googlicius/obsidian-steward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
