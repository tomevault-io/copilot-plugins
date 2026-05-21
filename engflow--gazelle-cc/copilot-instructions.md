## integration-tests

> How to run and write integration tests in this project

# Integration tests

## Running tests

- Use the command `bazel test //...` to run all tests.
- `example/` directory contains slow end-to-end tests that can only be run manually by the `.github/workflows/test_e2e.sh` script.

## Test structure

- `language/cc/testdata/` directory contains integration tests checking whether `gazelle_cc` produces expected Bazel rules for certain repository examples.
- Check out [generationtest.bzl](https://github.com/bazel-contrib/bazel-gazelle/blob/master/internal/generationtest/generationtest.bzl) to understand `gazelle_generation_test` macro
- Check out `bazel/gazelle_compilation_tests.bzl` to understand `gazelle_compilation_test` macro.
- All scenarios under `language/cc/testdata/` must be covered by `gazelle_generation_test`.
- By default, a scenario should be covered by `gazelle_compilation_test`, unless the scenario is not compilable by design, because of, e.g., unresolved `cc_library` dependencies.

## Adding integration tests

- In case of new `gazelle_cc` features, especially new/modified directives defined in `func (c *ccLanguage) KnownDirectives() []string`, add an appropriate test scenario under `language/cc/testdata/`.

---
> Source: [EngFlow/gazelle_cc](https://github.com/EngFlow/gazelle_cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
