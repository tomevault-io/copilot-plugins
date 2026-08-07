## skia-ex

> - Use the project Mix aliases; prefer `mix ci` for the full validation suite.

# Agent Guidelines

## Development

```sh
mix deps.get
mix ci
```

## Conventions

- Use the project Mix aliases; prefer `mix ci` for the full validation suite.
- GitHub CI uses the shared `elixir-vibe/actions/.github/workflows/elixir-rustler-ci.yml` workflow for Rustler setup and Cargo caching.
- Keep changes small, tested, and formatted.
- For generated native drawing code, follow `docs/codegen.md`: keep Skia semantics in Rusty Elixir modules under `Skia.Codegen.Rusty`, keep generated Rust builders under `Skia.Codegen.Rust`, and avoid raw Rust/body-string workarounds.

---
> Source: [dannote/skia_ex](https://github.com/dannote/skia_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
