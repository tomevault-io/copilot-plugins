## misonius

> > "Philosophy must be practical, not theoretical. Specs must be actionable, not ephemeral."

# CLAUDE.md — Musonius Development

> "Philosophy must be practical, not theoretical. Specs must be actionable, not ephemeral."
> — Musonius Rufus, 30–101 AD, "The Roman Socrates"

## What Is Musonius

Musonius is an open-source CLI tool that sits **between your intent and your AI coding agents**. It pre-computes codebase context, maintains persistent project memory, and generates optimized handoff documents so any downstream agent (Claude Code, Gemini CLI, Cursor, Aider, Codex) goes straight to surgical execution instead of burning tokens on exploration.

**Musonius is the coach, not the player.** It doesn't write code — it makes every tool that does write code 3–5x more effective.

**Domain:** musonius.dev
**License:** MIT
**Language:** Python 3.12+
**CLI framework:** Click or Typer (prefer Typer for type hints)

---

## Architecture Overview

### Core Components

```
musonius/
├── cli/                    # CLI entry points (Typer)
│   ├── __init__.py
│   ├── main.py             # musonius init | plan | prep | verify | memory
│   └── commands/
│       ├── init.py          # Index codebase, create .musonius/ scaffold
│       ├── plan.py          # Decompose task → phases via scout agent
│       ├── prep.py          # Generate agent-specific context file
│       ├── verify.py        # Cross-model adversarial review
│       └── memory.py        # View/edit project knowledge
├── indexer/                # Core engine — replaces 60% of token-wasting exploration
│   ├── parser.py           # Tree-sitter codebase parser
│   ├── graph.py            # Dependency graph builder
│   └── repomap.py          # Multi-level repo map generator (L0–L3)
├── memory/                 # Persistent project intelligence
│   ├── store.py            # SQLite store for conventions, decisions, failures
│   ├── schema.py           # Memory schema definitions
│   └── query.py            # Memory search and retrieval
├── context/                # The critical output layer
│   ├── generator.py        # Task + index + memory → context document
│   ├── budgets.py          # Token budget allocation and enforcement
│   ├── formats/
│   │   ├── claude.py       # CLAUDE.md generator (XML-style structured)
│   │   ├── gemini.py       # GEMINI.md generator (natural language)
│   │   ├── cursor.py       # .cursorrules generator
│   │   └── generic.py      # AGENTS.md fallback
│   └── serializer.py       # Compact serialization (65% smaller than JSON)
├── router/                 # Cost-aware model routing
│   ├── scout.py            # Free-tier scouting (Gemini Flash)
│   ├── planner.py          # Planning model selection
│   └── verifier.py         # Cross-model review routing
├── mcp/                    # MCP server for universal integration
│   └── server.py           # Exposes musonius_get_plan, _get_context, _verify, _memory_query
├── parallel/               # Git worktree parallel execution (v1.1+)
│   ├── worktree.py         # Worktree lifecycle management
│   ├── coordinator.py      # Phase dependency analysis + parallel dispatch
│   └── merger.py           # Conflict detection and resolution
└── utils/
    ├── tokens.py           # Token counting and budget tracking
    ├── git.py              # Git operations (tags, stash, rollback)
    └── config.py           # Configuration management
```

### Project Data Directory

```
.musonius/
├── constitution.md         # Immutable project rules (auto-generated on init)
├── sot/                    # Source of Truth — architecture decisions, API contracts
│   ├── TECH-001.md         # e.g., "Python 3.12+ with FastAPI"
│   └── API-001.md          # e.g., "All endpoints require OpenAPI docs"
├── epics/                  # Feature specs with requirements + acceptance criteria
│   └── {epic-id}/
│       ├── spec.md
│       └── phases/
│           ├── phase-01.md  # File-level instructions + code context
│           └── phase-02.md
├── memory/
│   ├── conventions.json    # Code patterns and standards
│   ├── decisions.json      # Architecture decisions with rationale
│   ├── failures.json       # Failed approaches (anti-pattern library)
│   └── graph.json          # Dependency graph cache
└── config.yaml             # User preferences, model config, autonomy levels
```

---

## CLI Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `musonius init` | Index codebase, create `.musonius/` scaffold, auto-detect conventions | `musonius init` |
| `musonius plan "task"` | Decompose task into phased plan via scout agent | `musonius plan "add rate limiting to public API"` |
| `musonius prep "task"` | Generate optimized context file for chosen agent | `musonius prep epic-004 --agent claude` |
| `musonius verify` | Capture git diff, cross-model adversarial review | `musonius verify --reviewer gemini` |
| `musonius memory` | View/edit/search project knowledge | `musonius memory search "auth"` |
| `musonius status` | Token usage, phase progress, memory stats | `musonius status` |
| `musonius rollback` | Restore to phase checkpoint | `musonius rollback epic-004 phase-2` |

---

## Key Design Principles

1. **Local-first, always.** Everything runs on the user's machine. No cloud dependency for context. No telemetry. No accounts.

2. **Agent-agnostic output.** Musonius generates markdown files that any tool can read. The handoff mechanism is always the same — optimized context files. It doesn't care what the downstream tool is.

3. **Token efficiency is the product.** Every design decision optimizes for fewer tokens consumed by downstream agents. The 7 optimization strategies:
   - AST-aware progressive context loading (4 levels: L0 signatures → L3 full implementation)
   - Compact serialization (65% smaller than JSON)
   - Hierarchical summarization with anchored compression
   - Scout/thinker separation with model tiering
   - Prompt caching and prefix optimization (30–50% cost reduction)
   - Tool schema pruning (1,000–2,000 tokens saved per call)
   - Diff-based verification (60–80% reduction vs full-file review)

4. **Memory compounds.** Every completed task enriches project memory. Conventions, decisions, failures, and patterns persist across sessions and across tools. A new developer running `musonius init` on a 6-month project gets instant tribal knowledge.

5. **Free tier first.** Route 60–70% of calls through free or near-free tiers (Gemini Flash for scouting and verification). Reserve premium model capacity for implementation and complex planning.

6. **Specs are durable artifacts, not ephemeral plans.** Everything in `.musonius/` is git-versioned, diffable, blameable, and PR-reviewable.

---

## Technology Stack

- **Python 3.12+** — primary language
- **Typer** — CLI framework (type-hinted, auto-docs)
- **tree-sitter** — AST parsing for dependency graph and repo map
- **SQLite** — memory store (via `sqlite3` stdlib, no ORM needed)
- **LiteLLM** — model routing to 100+ providers (Claude, Gemini, Ollama, OpenAI-compatible)
- **tiktoken / tokenizers** — token counting
- **GitPython** — git operations (worktrees, tags, diffs)
- **FastMCP** — MCP server implementation
- **Rich** — terminal output formatting
- **pytest** — testing
- **ruff** — linting and formatting

---

## Coding Standards

- **Type hints everywhere.** All function signatures must have complete type annotations. Use `from __future__ import annotations` in every file.
- **Docstrings on all public functions.** Google-style docstrings.
- **No classes where functions suffice.** Prefer pure functions. Use dataclasses or Pydantic models for data, not inheritance hierarchies.
- **Explicit over implicit.** No magic. If a function has side effects, name it clearly (`write_memory`, not `process`).
- **Error handling:** Use specific exceptions, never bare `except`. Wrap external calls (LLM APIs, file I/O, git) in try/except with meaningful error messages.
- **Tests:** Every module gets a corresponding `tests/test_{module}.py`. Test the contract, not the implementation. Use fixtures for SQLite and temp directories.
- **No print statements.** Use `rich.console` for user-facing output, `logging` for debug output.
- **Imports:** stdlib → third-party → local, separated by blank lines. No wildcard imports.

---

## Build Priority (v0.1 → Demo-Ready)

**Phase 1 (Days 1–2): Core Indexer**
- tree-sitter parser → dependency graph → multi-level repo map
- This is the engine. Without it, everything else is just a wrapper around raw LLM calls.
- Target: `musonius init` produces a `.musonius/` directory with constitution + graph

**Phase 2 (Day 3): Project Memory**
- SQLite store for conventions, decisions, failures
- Auto-populated from `musonius init` analysis
- Target: `musonius memory search "auth"` returns relevant decisions

**Phase 3 (Day 4): Context Generator**
- Task + index + memory → CLAUDE.md / GEMINI.md / AGENTS.md
- Agent-specific formatting (XML for Claude, natural language for Gemini, .cursorrules for Cursor)
- Prefix-optimized for prompt caching
- Target: `musonius prep "add rate limiting" --agent claude` outputs a surgical context file

**Phase 4 (Day 5): CLI + Scout Router**
- Wire up all commands
- Scout agent via Gemini CLI free tier for cheap exploration
- Cross-model verification via `musonius verify`
- Target: Full CLI workflow from init → plan → prep → verify

**Phase 5 (Days 6–7): Polish + Demo**
- Side-by-side comparison: raw Claude Code vs Musonius + Claude Code
- Show token usage difference, time difference, quality difference
- README, installation docs, demo GIF

---

## Critical Anti-Patterns (DO NOT)

- **DO NOT** build a web UI before the CLI works end-to-end
- **DO NOT** add LangChain, LlamaIndex, or any orchestration framework — Musonius IS the orchestration layer, built lean with direct API calls via LiteLLM
- **DO NOT** store any user code or context in the cloud
- **DO NOT** require an API key to run `musonius init` — local indexing must work with zero external dependencies
- **DO NOT** use raw SQL strings — use parameterized queries
- **DO NOT** catch and silence exceptions — log them, surface them
- **DO NOT** add features not on the build priority list until Phase 5 is complete
- **DO NOT** optimize for multiple languages simultaneously in v0.1 — start with Python (tree-sitter-python), expand later

---

## Model Routing Strategy

| Operation | Default Model | Fallback | Cost |
|-----------|--------------|----------|------|
| Scouting | Gemini Flash (free) | Local via Ollama | $0 |
| Planning | User's Claude Code subscription | Gemini Pro | Included |
| Implementation | Best available (Claude, Gemini, local) | — | Varies |
| Verification | Gemini Flash (free) — cross-model | Local model | $0 |
| Summarization | Local small model or Gemini Flash | — | $0 |

---

## MCP Server Tools (v0.1)

```
musonius_get_plan      → returns current phase plan with optimized context
musonius_get_context   → returns token-budgeted context for a file/function
musonius_verify        → triggers cross-model verification of current changes
musonius_memory_query  → searches project memory for decisions and patterns
musonius_record_decision → adds a new decision to Source of Truth
```

---

## Competitive Position

Musonius is NOT another coding agent. It is the **multiplier layer for the AI coding tools you already use.**

**Primary differentiators vs Traycer (closest competitor):**
- Open source (MIT) vs closed SaaS
- Persistent git-versioned memory vs ephemeral session-only specs
- Cross-model routing with free-tier exploitation vs single-provider premium-for-everything
- CLI-first + MCP + any IDE vs VS Code extension only
- Parallel execution via git worktrees vs sequential only
- 5-level autonomy control vs binary manual/YOLO
- Local-first, works offline vs cloud-required

**The hook:** "Stop burning tokens on exploration. Musonius pre-computes your codebase context locally so your AI agent goes straight to coding."

---

## File Naming Conventions

- All Python files: `snake_case.py`
- All config files: `snake_case.yaml` or `snake_case.json`
- All spec/doc files: `kebab-case.md` or `UPPER-CASE.md` for root docs
- Test files: `test_{module_name}.py`
- Epic IDs: `epic-{NNN}` (e.g., `epic-001`)
- SOT IDs: `{CATEGORY}-{NNN}` (e.g., `TECH-001`, `API-002`, `CONV-012`)
- Failure IDs: `FAIL-{NNN}` (e.g., `FAIL-003`)

---

## Git Conventions

- Branch naming: `feat/{short-description}`, `fix/{short-description}`, `refactor/{short-description}`
- Commit messages: conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`)
- Phase checkpoints: git tags `musonius/{epic-id}/{phase-N}-start`
- Never commit `.musonius/memory/` to the Musonius source repo (it's for user projects)
- Always commit `.musonius/constitution.md` and `.musonius/sot/` (they're project specs)

---

## Dependencies (pyproject.toml)

```toml
[project]
name = "musonius"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "typer>=0.12",
    "rich>=13.0",
    "tree-sitter>=0.23",
    "tree-sitter-python>=0.23",
    "litellm>=1.50",
    "tiktoken>=0.7",
    "gitpython>=3.1",
    "pydantic>=2.0",
    "fastmcp>=0.1",
    "pyyaml>=6.0",
]

[project.scripts]
musonius = "musonius.cli.main:app"
```

---

## Testing Strategy

- Unit tests for indexer (parser, graph, repomap) using fixture repos
- Unit tests for memory (CRUD, search, conflict detection)
- Unit tests for context generator (token budgets, format correctness)
- Integration tests for CLI commands (init → plan → prep → verify pipeline)
- Mock all LLM calls in tests — use recorded responses
- Target: 80%+ coverage on core modules before v0.1 release

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Musonius-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
