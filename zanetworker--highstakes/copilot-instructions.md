## highstakes

> Go CLI tool that identifies critical code paths using LLM blast radius analysis.

# Code Heatmap

Go CLI tool that identifies critical code paths using LLM blast radius analysis.

## Build and Test

```sh
go build -o heatmap ./cmd/highstakes
go test ./... -count=1
go install ./cmd/highstakes
```

## Project Structure

```
cmd/highstakes/main.go         CLI entry point (Cobra)
internal/
  analyzer/                  Static analysis (AST, imports, complexity)
  gitanalyzer/               Git history (commits, churn, contributors)
  llm/                       LLM blast radius assessment via OpenRouter
  scorer/                    Heat score formula and tier assignment
  prscore/                   PR risk scoring and circuit breakers
  incidents/                 Incident tracking persistence
  tui/                       Bubbletea TUI (tree model + view)
  dashboard/                 HTML dashboard generator (treemap + explorer)
  types/                     Shared type definitions
pkg/heatmap/                 Public API (generator, config, save/load)
docs/                        User-facing documentation
```

## Key Patterns

- Scoring: LLM blast radius (40%) + static analysis (30%) + git history (20%) + incidents (10%)
- LLM calls go through OpenRouter (OpenAI-compatible format), default model is DeepSeek V4 Flash
- Assessments cached in .heatmap/cache/ by SHA-256 of file content
- Analyzer excludes configurable in .heatmap/config.yaml (not hardcoded)
- All CLI commands support --json for machine-readable output
- Dashboard is self-contained HTML embedded as a Go string constant

## Conventions

- CLI verbs follow agentic CLI vocabulary: get, list, create, check (not show, find, add)
- Errors include valid values when an enum is the cause
- Exit codes: 0=success, 1=internal, 2=invalid input, 3=external failure, 4=not found
- Tests use table-driven patterns with t.Run subtests

---
> Source: [zanetworker/highstakes](https://github.com/zanetworker/highstakes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
