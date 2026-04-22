## baby-agent

> This repository is a chapter-based Go tutorial for building AI agents.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a chapter-based Go tutorial for building AI agents.
- `ch01`-`ch09`: runnable chapter implementations (each chapter evolves capabilities).
- `ch10`: placeholder for upcoming web/SSE chapter.
- `shared/`: common utilities (config, env loading, shared types, MCP helpers).
- `openspec/`: OpenSpec configuration.
- `.github/workflows/test-build.yml`: CI checks used for PR validation.

Follow the existing chapter pattern (`prompt.go`, `agent.go`, `main/main.go`, optional `tool/`, `context/`, `memory/`, `storage/`) when adding new functionality.

## Build, Test, and Development Commands
Use Go 1.25+.
- `cp .env.example .env`: initialize local configuration.
- `go run ./ch01/main --stream -q "hello"`: run a chapter entrypoint.
- `go test -v ./ch08/...`: run tests for one chapter.
- `go test -v ./...`: run all tests in the module.
- `go build -v ./ch09/main`: build a chapter binary (CI builds chapter mains).
- `gofmt -l .`: check formatting (matches CI).
- `gofmt -w <paths>`: apply formatting before commit.

## Coding Style & Naming Conventions
- Follow standard Go formatting and idioms (`gofmt` is required; tabs for indentation).
- Keep package names short, lowercase, and domain-focused (`context`, `tool`, `memory`).
- Exported identifiers: `PascalCase`; internal helpers: `camelCase`.
- Prefer small, composable files over large mixed-responsibility files.

## Testing Guidelines
- Place tests in `*_test.go` next to the code under test.
- Current tests focus on context policy modules (e.g., `ch05/context/policy_test.go`).
- Use descriptive test names like `Test<Feature>_<Scenario>`.
- For changes, add or update tests in the same chapter/package and run `go test -v ./<chapter>/...`.

## Commit & Pull Request Guidelines
- Keep commits scoped and imperative (patterns in history: `implement ch08`, `refactor ...`, `fix ...`).
- Optional prefixes are acceptable when useful (`feat:`, `fix:`, `refactor:`).
- PRs should include:
  - what changed and why,
  - affected chapters/packages,
  - local verification commands/results,
  - screenshots or terminal snippets for TUI-visible changes.

## Security & Configuration Tips
- Never commit secrets; keep keys in `.env` only.
- Use `.env.example` as the template for required variables (`OPENAI_BASE_URL`, `OPENAI_API_KEY`, `OPENAI_MODEL`).

---
> Source: [baby-llm/baby-agent](https://github.com/baby-llm/baby-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
