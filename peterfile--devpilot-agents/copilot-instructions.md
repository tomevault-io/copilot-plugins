## devpilot-agents

> Multi-agent orchestration framework for complex software engineering tasks.

# AGENTS.md

Multi-agent orchestration framework for complex software engineering tasks.

## Setup commands

- Install skills: `npx skills add PeterFile/devpilot-agents`
- Build wrapper: `cd codeagent-wrapper && go build -o codeagent-wrapper .`
- Run Go tests: `cd codeagent-wrapper && go test -v ./...`
- Run Python tests: `cd skills/multi-agent-orchestration/scripts && python -m pytest -v`

## Project structure

- `skills/multi-agent-orchestration/` - Core orchestration skill with Python scripts
- `skills/kiro-specs/` - Spec-driven workflow skill (requirements → design → tasks)
- `skills/test-driven-development/` - TDD skill
- `codeagent-wrapper/` - Go execution engine for parallel task dispatch
- `.opencode/agents/` - Agent definitions (king-arthur.md, gawain.md)

## Code style

### Go (codeagent-wrapper/)

- Go 1.21+
- Standard library preferred
- Error handling: return errors, don't panic
- Use `go fmt` and `go vet`

### Python (skills/\*/scripts/)

- Python 3.x
- Type hints encouraged
- Use pytest for testing
- Follow existing module patterns

## Testing

- Go: `go test -v ./...` in `codeagent-wrapper/`
- Python: `python -m pytest -v` in script directories
- Integration: `pytest test_e2e_orchestration.py`

## Key files

- `skills/multi-agent-orchestration/SKILL.md` - Orchestrator skill definition
- `skills/multi-agent-orchestration/scripts/orchestration_loop.py` - Main loop runner
- `skills/multi-agent-orchestration/references/agent-state-schema.json` - State schema

## Workflow

The orchestration workflow follows: init → dispatch → review → consolidate → sync

1. Parse Kiro spec (`tasks.md`) into `AGENT_STATE.json`
2. Dispatch tasks to Codex (code) or Gemini (UI) backends
3. Review changes with codex-review
4. Consolidate and sync to `PROJECT_PULSE.md`

## Notes / Gotchas

- `codeagent-wrapper --window-for` only affects tmux mode; it requires `--tmux-session`.
- OpenCode assignment agents (e.g. `gawain`) are tool-less; prompts must include `@path` file references (e.g. `@.kiro/specs/<spec>/tasks.md`) so the backend can see inputs.
- `skills/sequential-orchestration/scripts/sequential_loop.py` writes the dispatch unit **and its descendant task IDs** into `SEQUENTIAL_STATE.json` so dependency checks work.
- `skills/sequential-orchestration/runner.sh` is a thin wrapper over `skills/sequential-orchestration/scripts/sequential_loop.py` (for bash users).

---
> Source: [PeterFile/devpilot-agents](https://github.com/PeterFile/devpilot-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
