## researchagent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Google ADK (Agent Development Kit) project implementing "Forgetting Lab" - an autonomous AI scientist for researching catastrophic forgetting in neural networks. The agent runs iterative research cycles: hypothesis generation, critique, experimentation, and analysis.

## Development Commands

```bash
# Install dependencies
uv sync

# Run the agent via ADK CLI (from project root)
uv run adk run ika_agent

# Run with web interface (use "." not "ika_agent")
uv run adk web .

# Install Playwright browsers (required for computer use)
playwright install chromium
```

## Codebase Structure

```
agent1/
├── ika_agent/                    # Main agent package
│   ├── __init__.py              # Package init (exports root_agent)
│   ├── agent.py                 # Core agent definitions (~750 lines)
│   ├── playwright_computer.py   # Browser automation for computer use
│   └── .env                     # API keys (GEMINI_API_KEY)
├── outputs/                      # Persistent storage (outside agent package)
│   ├── memory.json              # Research history, insights, failures
│   ├── metrics.json             # Metrics tracking for trend analysis
│   ├── reports/                 # Generated papers and summaries
│   └── experiments/             # Versioned experiment snapshots
│       ├── index.json           # Experiment index for quick lookup
│       └── exp_XXX_XXXXXXXX/    # Per-experiment directories
│           ├── experiment.py    # Exact code that was run
│           ├── metadata.json    # Seeds, hyperparams, environment
│           └── results.json     # Execution output
├── .claude/
│   ├── settings.local.json      # Tool permissions
│   └── skills/                  # Project-local skills
│       ├── google-adk/
│       ├── gemini-genai/
│       ├── uv-advanced/
│       ├── prompt-engineering/
│       ├── mcp-builder/
│       └── headless-cli-agents/
├── pyproject.toml               # Dependencies: google-adk, playwright
├── main.py                      # Entry point (imports ika_agent)
├── CLAUDE.md                    # This file
├── .python-version              # Python version (uv)
└── .venv/                       # Virtual environment (uv managed)
```

## Architecture

### Agent Pipeline (`ika_agent/agent.py`)

The system uses Google ADK's SequentialAgent to chain specialized agents in a research cycle:

1. **Theorist** - Generates falsifiable hypotheses about catastrophic forgetting
2. **Critic** - Attacks hypotheses ruthlessly to find flaws
3. **Gatekeeper** - Makes PROCEED/REVISE/REJECT decisions
4. **Architect** - Designs minimal runnable experiments (has code executor)
5. **Analyst** - Provides honest numerical analysis of results
6. **Teacher** - Explains findings in simple terms
7. **Editor** - Reviews for scientific rigor
8. **Memory Saver** - Persists findings to disk

Additional standalone agents:
- **Reporter** - Generates scientific papers from accumulated research
- **Synthesizer** - Analyzes patterns across multiple research cycles
- **Browser Researcher** - Uses Playwright for web research (optional)

### Memory System

Persistent memory stored in `outputs/`:
- `memory.json` - Research history, insights, and failures
- `reports/` - Generated papers and summaries

The `MemoryManager` singleton provides thread-safe and process-safe access to memory with file-level locking.

### Experiment Versioning System

The `ExperimentVersionManager` saves reproducible experiment snapshots in `outputs/experiments/`:

Each experiment captures:
- **Code snapshot** - Exact Python code that was run (with SHA256 hash)
- **Random seeds** - Auto-extracted from code patterns
- **Hyperparameters** - Auto-extracted (lr, batch_size, epochs, etc.)
- **Environment** - Python, PyTorch, NumPy versions, git commit
- **Results** - Full execution output

Commands available through root agent:
- `list experiments` - Show all saved experiment versions
- `show experiment <id or cycle>` - Display experiment details
- `compare experiments <id1> <id2>` - Diff two experiments
- `rerun experiment <id>` - Get command to re-run

The Memory Saver agent automatically saves experiment versions after each successful cycle.

### Metrics Tracking System

The `MetricsTracker` singleton tracks experiment metrics over time for trend analysis:

- **Automatic extraction** - Parses accuracy, loss, forgetting, delta, baseline from experiment output
- **Tagging** - Tag experiments with method categories (ewc, replay, sam, etc.)
- **Trend analysis** - Track metrics over cycles, compute statistics
- **Progress dashboard** - Summary of research progress with trends

Commands available through root agent:
- `dashboard` / `progress` - Show research progress dashboard
- `trends <metric>` - Show trends for accuracy, loss, forgetting, etc.
- `tag experiment <id> <tags>` - Add tags to an experiment
- `find by tag <tag>` - Find experiments with a specific tag

Metrics are automatically recorded by the Memory Saver agent after each experiment.

### Hypothesis Novelty Checker

The `HypothesisSimilarityChecker` prevents redundant research by comparing new hypotheses against past ones:

- **Jaccard similarity** - Tokenized text comparison with stopword filtering
- **Warning thresholds** - Medium warning at 60%, high warning at 80% similarity
- **Cluster detection** - Identifies groups of similar research directions
- **Integrated with Theorist** - The Theorist agent automatically checks novelty before proposing

Commands available through root agent:
- `check novelty <hypothesis>` - Check if a hypothesis is similar to past ones
- `similarity matrix` - Show similarity relationships between all past hypotheses
- `health check` - Run system health check including hypothesis diversity analysis

### Data Backfill Utilities

Utilities to maintain data consistency and recover missing metrics:

- `backfill metrics` - Extract metrics from past cycle analyses that weren't tracked
- Automatic tagging based on hypothesis content (sam, swa, replay, normalization, etc.)

### Computer Use (`ika_agent/playwright_computer.py`)

Custom `PlaywrightComputer` implementation that wraps Playwright's sync API in a ThreadPoolExecutor to avoid Windows asyncio subprocess issues. Implements the ADK `BaseComputer` interface for browser automation.

### Models

**DO NOT CHANGE THE MODEL without user permission.**

- Standard agents: `gemini-3-flash-preview`
- Computer use agent: `gemini-2.5-computer-use-preview-10-2025` (no Gemini 3 version yet)

## Key Patterns

- All agents use `output_key` to pass data through the pipeline via session state
- Custom `FunctionTool` wrappers expose memory operations to agents
- The root `forgetting_lab` agent orchestrates cycles and handles user commands
- Research cycles auto-save via the `memory_saver` agent at the end of each cycle

## Available Skills

Claude Code has access to these skills for working on this project:

| Skill | Use For |
|-------|---------|
| `google-adk` | ADK architecture, SequentialAgent, transfer_to_agent, output_key/session.state flow, FunctionTool patterns, common pitfalls |
| `gemini-genai` | Gemini API, model IDs, thinking levels, structured outputs, function calling, streaming, multimodal |
| `uv-advanced` | Python environment management, uv sync/run/add, dependency resolution |
| `prompt-engineering` | Agent instruction crafting, chain of thought, XML structuring |
| `mcp-builder` | Building MCP servers for tool integrations |
| `headless-cli-agents` | CLI automation, multi-agent orchestration patterns |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cuba6112)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cuba6112)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
