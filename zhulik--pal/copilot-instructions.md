## tests

> Tests must always be defined in `_test` package and only test package's public API.

# Tests

Tests must always be defined in `_test` package and only test package's public API.

When testing functions, each function must have a separate Test function, for instance:
Function `Foo` has test `TestFoo(t *testing.T)`. Each test case is defined inside it in a separate subtest with `t.Run()`.

When testing structs, every method of the struct must have a separate Test function, for instance:
Struct `Foo` has a method `Bar`, tests for this method should be defined in `TestFoo_Bar(t *testing.T)`.
Each test case is defined inside it in a separate subtest with `t.Run()`.

Read-only resources which can be reused across multiple tests, should be defined as global variables.
Resources which can be reused across multiple tests files, must be defined in common_test.go

For asserts and mocks `github.com/stretchr/testify` should be used.

Make sure all code paths are tested.

---
> Source: [zhulik/pal](https://github.com/zhulik/pal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
