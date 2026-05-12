## yargo

> Pure Go implementation of YARA, eliminating the need for go-yara/cgo dependencies. Currently provides a parser; a scanner using the Aho-Corasick algorithm is planned.

# Yargo

Pure Go implementation of YARA, eliminating the need for go-yara/cgo dependencies. Currently provides a parser; a scanner using the Aho-Corasick algorithm is planned.

## Instructions

After modifying any Go file, run `gofumpt -w <filename>` to format it.

When implementing features, write the tests first, then write the implementation. After completing the implementation, run `go test ./...` to verify.

When writing Go code, prefer early returns over nested conditionals. Keep the happy path to the left with minimal indentation.

Never use `go build`. Use `go run .` instead.

To swap the regex library, only change the import path and keep the `regexp` alias. For example, to switch from coregex to go-re2, change `regexp "github.com/coregx/coregex"` to `regexp "github.com/wasilibs/go-re2"`.

---
> Source: [sansecio/yargo](https://github.com/sansecio/yargo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
