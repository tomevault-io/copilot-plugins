## testing-tsconfig

> When adding new tests to this project, always use the existing [tsconfig.basic.json](mdc:test/tsconfig.basic.json) fixture. Do not create new tsconfig files for testing purposes.


# Testing TypeScript Configuration

When adding new tests to this project, always use the existing [tsconfig.basic.json](mdc:test/tsconfig.basic.json) fixture. Do not create new tsconfig files for testing purposes.

The test configuration is already set up in [tsconfig.basic.json](mdc:test/tsconfig.basic.json) and should be reused for all new test cases. This maintains consistency across the test suite and avoids unnecessary configuration duplication. Do not create any new source files within `test`. Do not create an new entrypoints.

---
> Source: [colinhacks/zshy](https://github.com/colinhacks/zshy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
