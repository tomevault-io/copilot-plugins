## deepagent-research-context-engineering

> **Generated:** 2026-01-13

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-01-13

---

## OVERVIEW

Multi-agent research system demonstrating **FileSystem-based Context Engineering** using LangChain's DeepAgents framework. Includes Python orchestrator, Rust port via Rig framework, and Next.js chat UI.

---

## STRUCTURE

```
/
  research_agent/          # Python DeepAgent orchestrator (see AGENTS.md)
  context_engineering_research_agent/  # Extended agent with 5 strategies
  deep-agents-ui/          # Next.js React frontend (see AGENTS.md)
  rust-research-agent/     # Rust implementation (see AGENTS.md)
    rig-deepagents/        # Pregel-based middleware runtime
    rig-rlm/               # Recursive Language Model agent
  tests/                   # pytest test suite (see AGENTS.md)
  skills/                  # Project-level skills (SKILL.md per skill)
  research_workspace/      # Agent output directory (ephemeral)
  deepagents_sourcecode/   # Vendor: upstream library reference
```

---

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Modify orchestrator | `research_agent/agent.py` | SubAgent assembly, tools, middleware |
| Add research tool | `research_agent/tools.py` | `tavily_search`, `think_tool` |
| Autonomous researcher logic | `research_agent/researcher/` | Three-phase workflow |
| Context strategies | `context_engineering_research_agent/context_strategies/` | 5 patterns |
| Frontend components | `deep-agents-ui/src/app/components/` | Chat UI |
| Rust Pregel runtime | `rust-research-agent/rig-deepagents/src/pregel/` | Graph execution |
| Rust middleware | `rust-research-agent/rig-deepagents/src/middleware/` | Tool injection |
| Add new skill | `skills/{skill-name}/SKILL.md` | YAML frontmatter + instructions |

---

## CONVENTIONS

### Deviations from Standard Patterns

- **Backend factory pattern**: Always use `backend_factory(rt: ToolRuntime)` - middleware depends on this signature
- **SubAgent naming**: Use `researcher`, `explorer`, `synthesizer` - hardcoded in prompts
- **File paths**: Paths starting with "/" route to `research_workspace/`; others are in-memory
- **Korean comments**: Docstrings and some comments in Korean (bilingual codebase)

---

## ANTI-PATTERNS

- **DO NOT** commit `.env` files (contains API keys)
- **DO NOT** instantiate researcher directly - use `get_researcher_subagent()`
- **DO NOT** skip `think_tool()` between searches - explicit reflection required
- **NEVER** write raw JSON to user - always format responses (see `deepagents_cli/tools.py`)
- **NEVER** lie to exit early - complete TODO items fully (see `researcher/runner.py`)

---

## COMMANDS

### Python Development

```bash
uv sync                           # Install dependencies
langgraph dev                     # Start backend (port 2024)
uv run ruff format .              # Format code
uv run ruff check .               # Lint
uv run mypy .                     # Type check
uv run pytest tests/              # Run tests
```

### Frontend Development

```bash
cd deep-agents-ui
yarn install && yarn dev          # Dev server (port 3000)
yarn build                        # Production build
yarn lint && yarn format          # Lint + format
```

### Rust Development

```bash
cd rust-research-agent/rig-deepagents
cargo test                        # Run tests (159)
cargo clippy -- -D warnings       # Lint (strict)
cargo build --features checkpointer-sqlite  # Build with features
```

---

## ENVIRONMENT VARIABLES

Copy `env.example` to `.env`:

| Variable | Required | Purpose |
|----------|----------|---------|
| `OPENAI_API_KEY` | Yes | gpt-4.1 model |
| `TAVILY_API_KEY` | Yes | Web search |
| `LANGSMITH_API_KEY` | No | Tracing (`lsv2_pt_...`) |
| `ANTHROPIC_API_KEY` | No | Claude models + caching |

---

## SUBDIRECTORY KNOWLEDGE

See AGENTS.md files in:
- `research_agent/AGENTS.md` - Orchestrator details
- `context_engineering_research_agent/AGENTS.md` - Context strategies
- `deep-agents-ui/AGENTS.md` - Frontend architecture
- `tests/AGENTS.md` - Test organization
- `rust-research-agent/AGENTS.md` - Rust overview (3 tiers)
- `rust-research-agent/rig-deepagents/AGENTS.md` - Middleware architecture
- `rust-research-agent/rig-rlm/AGENTS.md` - Recursive LLM pattern

---
> Source: [HyunjunJeon/Deepagent-research-context-engineering](https://github.com/HyunjunJeon/Deepagent-research-context-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
