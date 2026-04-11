## network-monitor

> Source code centers on `main.go` with feature logic under `internal/`. Key packages: `internal/monitor` for worker orchestration, `internal/ping` for ICMP sampling, `internal/database` for SQLite, and `internal/web` for the dashboard. Static assets live in `static/`, build artefacts in `build/`, generated reports in `reports/`, and shared automation aids in `llm-shared/`.

# Agent Handbook

## Project Structure & Core Modules

Source code centers on `main.go` with feature logic under `internal/`. Key packages: `internal/monitor` for worker orchestration, `internal/ping` for ICMP sampling, `internal/database` for SQLite, and `internal/web` for the dashboard. Static assets live in `static/`, build artefacts in `build/`, generated reports in `reports/`, and shared automation aids in `llm-shared/`.

## Build, Test, and Runtime Commands

Run `task build` before claiming work complete; it wraps linting, tests, and the build. Use `task build-linux` for cross-compiles, `task lint` to enforce formatting/vetting, `task clean` to reset `build/`, and `task dev` (or `go run . --dev`) for the live dashboard. For containers, prefer `docker-compose up --build` / `docker-compose down`.

## Git & Workflow Norms

- Never commit directly to `main`/`master`; develop on feature branches and keep commits focused.
- Rebase before merge, rely on pull requests for review, and close the loop with linked tracking issues.
- When working from checklists, tick items as you go and treat a task as complete only once `task build` passes and tests exist for the new logic.

## Coding Style & Language Guidance

Target Go 1.21 in this repo, but align with the broader Go practices in `llm-shared/languages/go.md`: prefer the latest stable Go toolchain (currently 1.24), use `goimports -w .` for formatting/imports, justify third-party deps, and lean on standard library packages. Keep packages lower_snake_case, exported identifiers in PascalCase, and split oversized files across focused modules.

## Testing Expectations

Collocate tests with code (e.g. `internal/ping/ping_test.go`). Run `task test` or `go test ./...` prior to push; CI mirrors this through `task test-ci`. Write table-driven tests with descriptive names like `TestWorkerHandlesTimeout`, and ensure even small changes land with basic coverage.

## Commit & PR Conventions

Use Conventional Commits (`feat:`, `fix:`, `refactor:`). Summarise monitoring/UI impact in PRs, attach screenshots or CLI output when user-facing, and audit docs whenever touching pieces like `Taskfile.yml` or `static/`.

## LLM Shared Resources

- `llm-shared/README.md` points to shared playbooks. Review it at the start of an engagement.
- `llm-shared/project_tech_stack.md` covers repository hygiene: branch policy, Gemini CLI usage for large-code analysis, and the `validate-docs` workflow.
- `llm-shared/GITHUB.md` documents `gh` CLI flows for managing issues/labels; bootstrap recommended labels with `./llm-shared/create-gh-labels.sh`.
- `llm-shared/shell_commands.md` standardises shell tooling (`rg`, `fd`, etc.).
- Language primers in `llm-shared/languages/` hold deeper Go/Python/JavaScript notes; consult `go.md` for dependency policy, lint setup, and CI expectations.

## Shared Utilities & Templates

Automation helpers live under `llm-shared/utils/`: the Go/Python/JS function listers (`gofuncs`, `pyfuncs`, `jsfuncs`) aid rapid code discovery, while `validate-docs` checks repo structure. Template assets in `llm-shared/templates/` supply starter configs for Taskfiles, CI workflows, changelogs, and language-aware `.gitignore` variants.

## Shell & Tooling Expectations

Prefer the modern command set: `rg` over `grep`, `fd` over `find`, and embrace context-friendly flags (see `llm-shared/shell_commands.md`). Respect `.gitignore` defaults, use glob filters, and rely on these tools for fast code searches and batch operations.

## Operations & Configuration Notes

Runtime configuration comes from CLI flags and files in `config/`. SQLite state lives in `network_monitor.db`; keep database files out of commits. Before `task report`, ensure `build/network-monitor` exists. Coordinate schema or retention updates with reporting charts so exported PNGs stay accurate. Any running web service must expose `/whoami` with project metadata, and existing processes should be identified via that endpoint before restarting or terminating them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lepinkainen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lepinkainen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
