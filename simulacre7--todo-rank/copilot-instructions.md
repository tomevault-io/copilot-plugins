## todo-rank

> `todo-rank` is a CLI tool for scanning and prioritizing TODO/FIXME/@next comments in codebases.

# Cursor Rules for todo-rank

## Tool Overview

`todo-rank` is a CLI tool for scanning and prioritizing TODO/FIXME/@next comments in codebases.

## Installation

```bash
go install github.com/simulacre7/todo-rank/cmd/todo-rank@latest
```

Requires `$HOME/go/bin` in PATH.

## Usage Patterns

### Check priorities before coding
```bash
todo-rank --min-score 80
```

### Generate markdown task list
```bash
todo-rank --format md --out TODOs.md
```

### Scan specific directory
```bash
todo-rank --root ./src
```

### Show all TODOs
```bash
todo-rank --min-score 0
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| --root | Scan directory | . |
| --ignore | Skip directories (csv) | .git,node_modules,dist |
| --format | Output: text or md | text |
| --out | Output file | stdout |
| --min-score | Minimum score | 0 |
| --tags | Tags to scan (csv) | TODO,FIXME,@next |

## Score Reference

**Priority Levels:**
- P0 (Now): score >= 120
- P1 (Soon): score >= 80
- P2 (Later): score >= 40
- P3 (Cleanup): score < 40

**Score Components:**
- FIXME: +100, TODO: +50, @next: +30
- [P0]: +100, [P1]: +70, [P2]: +40, [P3]: +10
- cmd/ or main.go: +20
- *_test.*: -20

## Recommended Workflow

1. Run `todo-rank --min-score 80` at session start
2. Address P0 items before new features
3. Use `--format md` for user-facing output
4. Check module-specific TODOs with `--root`

## Comment Format

```
// TODO: basic task
// TODO[P1]: prioritized task
// FIXME[P0]: critical fix needed
// @next: next sprint item
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simulacre7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
