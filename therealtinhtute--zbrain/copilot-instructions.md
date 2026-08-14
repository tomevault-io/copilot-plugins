## zbrain

> zbrain is a Go-native CLI for local-first trusted memory. Core layout:

# Repository Guidelines

## Project Structure & Module Organization

zbrain is a Go-native CLI for local-first trusted memory. Core layout:

- `cmd/zbrain/` — CLI binary entrypoint.
- `internal/cli/` — command dispatch and user-facing command behavior.
- `internal/runtime/` — runtime paths, config, embedded asset extraction, workspace layout, OKF claim concepts, evidence, index, and trusted query logic.
- `assets/` — runtime content embedded into the binary and copied by `zbrain setup`.
- `docs/` — durable plans and supporting project documentation.

Keep command handlers thin: parse args, call runtime logic, and print JSON or user-facing text. Keep durable runtime behavior in `internal/runtime/`.

## Build, Test, and Development Commands

```bash
go test ./...                       # run all tests
make test                           # same test gate
make build                          # build dist/zbrain
make smoke                          # build and run isolated lifecycle smoke
```

Smoke tests must use `ZBRAIN_HOME` so they never touch real runtime data.

## Coding Style & Naming Conventions

- Go 1.24.
- Use standard Go formatting.
- Prefer small package-level functions and structs that match surrounding code.
- Keep filesystem writes behind explicit runtime stores or CLI commands.
- Preserve `assets/` as the runtime content source of truth.

## Testing Guidelines

- Add focused `*_test.go` coverage next to the package being changed.
- Use temp directories and explicit `ZBRAIN_HOME` isolation.
- Run `go test ./...` before claiming completion.
- For command/runtime behavior, also run a relevant isolated smoke command.

## Asset Authoring Guidelines

- Skill files in `assets/skills/*/SKILL.md` must have frontmatter: `name`, `description`, `version`.
- Engine files in `assets/engine/` are plain Markdown.
- Template files in `assets/templates/` use `{{placeholder}}` tokens matching the Go scaffold logic.
- Trusted claim templates should be OKF-style Markdown with `type: zbrain.claim` and `zbrain.profile: zbrain.trusted-memory/v1`.
- Evidence metadata templates must match the Go runtime `source.yaml` shape.
- After editing `assets/`, run tests and smoke; assets are embedded directly by Go.

## Security & Configuration Tips

- Do not commit secrets, personal workspace data, or populated runtime output.
- Workspace isolation is a hard rule; never read across workspace boundaries unless the caller passed explicit `--include`.
- Evidence snapshots are immutable local copies; never mutate a captured source after creation.
- Raw evidence is untrusted source data. Only approved OKF claim concepts are trusted context.
- Drafts are promotion candidates, not answer material.
- Derived SQLite indexes are disposable and must be rebuildable from canonical Markdown.

## Commit & Pull Request Guidelines

Use Conventional Commit style, matching recent history such as `feat(cli): ...`, `fix(runtime): ...`, and `docs(spec): ...`. Keep scopes specific to the area changed. PRs should include a short summary, affected paths, and commands run to verify.

<!-- ZHARNESS:BEGIN -->
## Harness

Run `zharness --version`, then `zharness preflight <stage> [--mode <mode>] --json` for every workflow skill invocation. Follow a returned stop and recovery exactly.

Read `docs/WORKFLOW.md`, then only the returned stage playbook and repository material relevant to the requested outcome. Repository docs, code, tests, and observable behavior are authoritative; the database is a lifecycle ledger and recovery index.

Read-only and bounded work may use reduced mode and must not mutate harness state. Durable planning, full execution, full checks, and durable handoffs require an initialized database. Claim completion only with executable or observable evidence.
<!-- ZHARNESS:END -->

---
> Source: [therealtinhtute/zbrain](https://github.com/therealtinhtute/zbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
