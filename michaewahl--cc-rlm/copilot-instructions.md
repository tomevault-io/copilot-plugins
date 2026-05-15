## cc-rlm

> REPL-based context engine for Claude Code.

# CC-RLM

REPL-based context engine for Claude Code.
Stack: Claude Code → CCR (proxy) → RLM Gateway (REPL brain) → Ollama (qwen2.5-coder:7b).

## What this does

Instead of dumping the whole repo into tokens, the RLM layer:

1. Maintains a live import graph of the repo (incremental, mtime-aware)
2. Runs walkers (Python AST, TS/JS regex, git diff) — results cached by mtime
3. Slices files at symbol level (function/class bodies, not arbitrary line ranges)
4. Skips files Claude already saw this session if they haven't changed
5. Hands a < 8K token context pack to the model as a precompiled header

## Prompt-driven routing

Prefix any message to override where it goes:

| Prefix      | Destination                                              |
| ----------- | -------------------------------------------------------- |
| `/claude …` | Anthropic API (full Claude reasoning)                    |
| `/local …`  | Ollama direct, no context enrichment                     |
| `/repo …`   | Force RLM enrichment                                     |
| (none)      | Auto: code signal → Ollama+RLM, knowledge Q → Anthropic  |

## Layout

```
ccr/          proxy — route, auth, fallback         → ccr/CLAUDE.md
rlm/          REPL brain — walkers, context pack     → rlm/CLAUDE.md
rlm/walkers/  subprocess walker scripts              → rlm/walkers/CLAUDE.md
tests/eval/   eval harness — token reduction + recall
docs/adr/     why decisions were made
.claude/      hooks (UserPromptSubmit, PreToolUse)
```

## Key constraints

- Walker timeout: 500ms each (configurable via RLM_WALKER_TIMEOUT_MS)
- Context pack hard cap: 8K tokens (configurable via RLM_TOKEN_BUDGET)
- Walkers are stateless subprocess scripts — they print JSON and exit
- Walker results cached by file mtime; RepoIndex keeps import graph in memory
- Session dedup: unchanged files skipped on turns 2+ (32% additional savings)

## Architecture decisions

- [ADR-001](docs/adr/001-repl-over-rag.md) — why REPL walkers, not vector search
- [ADR-002](docs/adr/002-subprocess-walkers.md) — why subprocess isolation
- [ADR-003](docs/adr/003-token-budget.md) — why 8K cap, not bigger

## Running locally (no Docker)

```bash
poetry install
poetry run uvicorn rlm.main:app --port 8081   # RLM
poetry run uvicorn ccr.main:app --port 8080   # CCR
```

Point Claude Code at CCR: `ANTHROPIC_BASE_URL=http://localhost:8080`

---
> Source: [michaewahl/CC-RLM](https://github.com/michaewahl/CC-RLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
