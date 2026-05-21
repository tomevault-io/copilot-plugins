## build

> What's the build structure of the project and how to traverse through the code

# Build structure

- `gazelle_cc` is a Go project built with Bazel and consists mostly of `go_library` and `go_test` rules.
- `gazelle_cc` is an extension implementing [`language.Language`](https://pkg.go.dev/github.com/bazelbuild/bazel-gazelle/language#Language) interface built together with the core [`gazelle`](https://github.com/bazel-contrib/bazel-gazelle) by [`gazelle_binary`](https://github.com/bazel-contrib/bazel-gazelle/blob/master/internal/gazelle_binary.bzl) macro.
- Scan the `$(bazel info output_base)/external` directory to see the exact checked out versions of dependencies specified in the `MODULE.bazel` file.

---
> Source: [EngFlow/gazelle_cc](https://github.com/EngFlow/gazelle_cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
