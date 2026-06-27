## gputrace

> - Build the CLI with `go install ./cmd/gputrace` or `make reinstall` on macOS when you need the bundled app flow.

# Agent Guidelines

- Build the CLI with `go install ./cmd/gputrace` or `make reinstall` on macOS when you need the bundled app flow.
- Validate changes with `go test ./...` and `go vet ./...`.
- Keep public user documentation in `README.md` and `docs/`.
- Put reverse-engineering notes and implementation archaeology in `docs/research/`.
- Keep unsupported experiments and local-only tooling out of the published repo.
- Do not commit local binaries, trace dumps, or generated screenshots unless they are intentional test fixtures.

---
> Source: [tmc/gputrace](https://github.com/tmc/gputrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
