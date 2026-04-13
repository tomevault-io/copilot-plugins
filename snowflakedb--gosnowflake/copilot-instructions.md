## gosnowflake

> 1. Unless it's necessary or told otherwise, try reusing existing files, both for implementation and tests.


# Go files rules

## General

1. Unless it's necessary or told otherwise, try reusing existing files, both for implementation and tests.
2. If possible, try running relevant tests.

## Tests

1. Create a test file with the name same as prod code file by default.
2. For assertions use our test helpers defined in assert_test.go.

## Logging

1. Add reasonable logging - don't repeat logs, but add them when it's meaningful.
2. Always consider log levels.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/snowflakedb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
