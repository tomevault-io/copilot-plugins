## tagpr

> `docs/site` is a Hugo module, not a Go source module.

# Contributor instructions

## Hugo site dependencies

`docs/site` is a Hugo module, not a Go source module.

- Never run `go mod tidy` in `docs/site`. It removes Hugo theme dependencies,
  such as Hextra, because they are not Go packages.
- Use `make docs-deps` or run `hugo mod tidy` from `docs/site` instead.
- Running `go mod tidy` at the repository root is safe; it does not traverse
  into the nested `docs/site` module.

---
> Source: [Songmu/tagpr](https://github.com/Songmu/tagpr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
