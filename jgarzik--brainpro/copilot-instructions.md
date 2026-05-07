## brainpro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
cargo build --release          # Build all binaries
cargo test                     # Run all tests
cargo test circuit_breaker     # Run specific module tests
cargo test --no-fail-fast -- --nocapture  # Verbose test output
```

## Manual Validation Suite (COSTS MONEY)

**DO NOT RUN without explicit user request** - these tests call real LLM APIs.

```bash
cd validation && pip install -r requirements.txt

# Run all tests (~$1.25)
pytest

# Run specific category
pytest tests/test_01_tools.py       # Basic tools
pytest tests/test_05_agent_loop.py  # Core multi-turn
pytest tests/test_06_plan_mode.py   # Core plan workflow

# Run with different modes
pytest --mode=yo        # Direct binary (default)
pytest --mode=native    # Native gateway
pytest --mode=docker    # Docker gateway
```

Tests validate outcomes (file creation, tool invocation, semantic content) not exact LLM output. Priority order for quick validation: `test_01_tools` → `test_05_agent_loop` → `test_06_plan_mode`.

See `validation/README.md` for details.

**Binaries produced:**
- `yo` - Direct CLI (MrCode persona)
- `brainpro-gateway` - WebSocket gateway server
- `brainpro-agent` - Agent daemon for gateway mode

## Architecture Overview

### Two Execution Paths

1. **Direct (`yo`)**: Single binary CLI using MrCode persona (7 tools). Interactive REPL or one-shot mode.
2. **Gateway**: `brainpro-gateway` + `brainpro-agent` daemon using MrBot persona (12+ tools). Supports WebSocket clients, yield/resume approval flows.

### Core Flow

```
User → CLI/Gateway → Agent Loop (agent.rs) → LLM Backend + Policy Engine
```

The agent loop (`src/agent.rs`) runs turn-based: prompt → LLM → tool calls → policy check → execute → repeat (max 12 iterations).

### Key Modules

| Module | Purpose |
|--------|---------|
| `agent.rs` | Core agent loop, tool orchestration |
| `cli.rs` | REPL, slash commands |
| `policy.rs` | Permission engine (allow/ask/deny rules) |
| `backend.rs` | Lazy-loaded backend registry |
| `llm.rs` | HTTP client with jittered backoff, SecretString credentials |
| `circuit_breaker.rs` | Closed→Open→HalfOpen state machine per backend |
| `provider_health.rs` | Health tracking (Healthy/Degraded/Unhealthy) |
| `privacy.rs` | ZDR privacy levels, sensitive pattern detection |
| `model_routing.rs` | Task-based model selection by category |
| `persona/` | Persona loader, MrCode and MrBot implementations |
| `tools/` | Individual tool implementations |

### Persona System

Personas are defined in `config/persona/{name}/`:
- `manifest.md` - YAML frontmatter with tool list, assembly order
- `identity.md` - Who the agent is
- `tooling.md` - Tool usage instructions
- `soul.md` - Personality (MrBot only)
- `plan-mode.md`, `optimize.md` - Conditional prompt sections

### Protocol Layers

- **Client ↔ Gateway**: WebSocket, JSON-RPC style, port 18789
- **Gateway ↔ Agent**: Unix socket (`/run/brainpro.sock`), NDJSON streaming

### LLM Backend Abstraction

All backends use OpenAI-compatible `/v1/chat/completions` format. Target format: `model@backend` (e.g., `claude-3-5-sonnet-latest@claude`).

Built-in backends: Venice (default), OpenAI, Anthropic, Ollama.

### Extension Points

- **Subagents**: `.brainpro/agents/<name>.toml` - restricted tool sets
- **Skill Packs**: `.brainpro/skills/<name>/SKILL.md` - reusable instructions
- **Custom Commands**: `.brainpro/commands/<name>.md` - user slash commands
- **MCP Servers**: External tool servers via config

## Configuration

Config files (priority order):
1. CLI arguments
2. `.brainpro/config.local.toml` (git-ignored)
3. `.brainpro/config.toml` (project)
4. `~/.brainpro/config.toml` (user)

API keys via environment: `VENICE_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`

---
> Source: [jgarzik/brainpro](https://github.com/jgarzik/brainpro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
