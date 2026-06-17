## joern-skill

> Deep code analysis using Joern CPG. Analyze dataflow, blast radius, callers/callees instantly. Use when understanding code structure, planning refactors, or assessing change impact.


# Joern Code Analysis

Deep code analysis using Joern CPG (Code Property Graph).
Understand code structure, dependencies, and impact of changes **instantly**.

## What is Joern?

Joern builds a **Code Property Graph** — a pre-computed graph combining AST, CFG, and data flow.
After initial setup (~30 sec), all queries are **instant** (no more grep/find scanning).

## Dependencies & Setup

**Required:**
- Python 3.10+
- Java 21 (OpenJDK)
- Joern CLI

**Python Libraries:**
- `requests` - HTTP client for Joern server

See `references/setup.md` for complete installation instructions.

## Quick Start

```bash
# 1. Install (one-time)
brew install joern openjdk@21
export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"

# 2. Create CPG for your project (~30 sec)
uv run python -m scripts.joern create .

# 3. Analyze!
uv run python -m scripts.joern --cpg landing dataflow scripts/your/module
```

## Core Commands

### Business Dataflow Analysis

Generate detailed pipeline diagram with docstrings, state mutations, and cross-step data flow:

```bash
uv run python -m scripts.joern --cpg landing dataflow scripts/adw/decompose
uv run python -m scripts.joern --cpg landing dataflow scripts/adw/decompose --entry run_decompose_v3
```

**Output shows:**
- Steps with docstrings (from Python source)
- Inputs — function parameters + state reads (`get_*`)
- Writes — state mutations (`set_*`)
- External markers — `◀── File`, `◀── Subprocess`, `◀── LLM`, `◀── DB`
- Cross-Step Data Flow — which data affects multiple steps
- Blast radius — who calls this, what files affected

### Blast Radius

Before changing a function, understand the impact:

```bash
uv run python -m scripts.joern --cpg landing blast_radius get_cursor
# → {direct_callers: 47, affected_files: 12, risk: "high"}
```

### Find Callers/Callees

```bash
uv run python -m scripts.joern --cpg landing callers spawn_worker --depth 2
uv run python -m scripts.joern --cpg landing callees run_orchestrator
```

### Methods in Module

```bash
uv run python -m scripts.joern --cpg landing methods_in scripts/adw/decompose/
```

### Dead Code Detection

```bash
uv run python -m scripts.joern --cpg landing dead_code scripts/adw/tools/
# Warning: May have false positives (API endpoints, callbacks)
```

## CPG Management

```bash
# List available CPGs
uv run python -m scripts.joern list

# Switch to different CPG
uv run python -m scripts.joern use KeepstarOne

# Server management
uv run python -m scripts.joern server status
uv run python -m scripts.joern server start
uv run python -m scripts.joern server stop
```

## Why Joern?

| Without Joern | With Joern |
|--------------|------------|
| `grep -r "function"` → 10 sec | `blast_radius("function")` → instant |
| Manual tracing of calls | Full call graph in CPG |
| Guessing impact | Precise blast radius |
| Re-scanning on every query | Pre-computed graph |

## Limitations

1. **CPG gets stale** — Recreate after major code changes
2. **Python focused** — Queries optimized for Python (Joern supports 13 languages)
3. **False positives in dead_code** — API endpoints, callbacks appear unused

## Files

```
scripts/joern/
├── __init__.py      # Exports
├── cli.py           # CLI interface
├── client.py        # HTTP client for Joern server
├── dataflow.py      # Business dataflow analyzer
├── processor.py     # Response formatting
├── queries.py       # High-level query functions
├── server.py        # Server management
└── cpg/             # CPG storage
```

See `references/reference.md` for Python API documentation.

---
> Source: [Starknight-one/joern-skill](https://github.com/Starknight-one/joern-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
