## kube2docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

kube2docs — a Python CLI that crawls a Kubernetes cluster and produces structured operational knowledge (JSON profiles, dependency graphs), with optional AI-powered documentation generation.

## Commands

```bash
# Install (editable, with dev deps)
uv pip install -e ".[dev]"

# Run tests
python -m pytest
python -m pytest tests/test_schemas.py -v                              # one file
python -m pytest tests/test_schemas.py::TestContainerInfo::test_create_minimal -v  # one test

# Lint, format & type check
ruff check src/
ruff format src/
mypy src/kube2docs/

# CLI entry point (after install)
kube2docs scan --kubeconfig ~/.kube/config --output ./kb/
kube2docs generate --input ./kb/ --output ./docs/ --model openrouter/anthropic/claude-haiku-4-5

# Agentic scan (LLM-driven Phase 3 instead of deterministic Phase 2)
kube2docs scan --kubeconfig ~/.kube/config --output ./kb/ \
  --agentic --model openrouter/moonshotai/kimi-k2.5 --api-key $OPENROUTER_API_KEY
```

## Output Directories

`kb/`, `docs/`, and `output/` are gitignored scan/generation output directories. They may be absent or empty in a fresh clone.

## AI Provider Configuration

The `generate` command uses litellm, supporting multiple backends. Set the appropriate env var (see `.env.example`):
- `ANTHROPIC_API_KEY` — Anthropic native
- `OPENAI_API_KEY` — OpenAI
- `OLLAMA_API_BASE` — local Ollama

Or pass `--api-key` on the CLI.

## Architecture

Pipeline with a default boundary: **scanning is deterministic (no AI), generation uses AI**. An opt-in `--agentic` flag replaces the deterministic deep-inspect phase with an LLM-driven one.

### Scanning (`kube2docs scan`)
1. **Survey phase** (`phases/survey.py`) — read-only K8s API calls; inventories Deployments, StatefulSets, DaemonSets, CronJobs, Jobs, Pods, Services, Ingresses, NetworkPolicies, ConfigMaps, Secrets. Outputs per-workload JSON profiles. Always runs.
2. **Deep inspect phase** (`phases/deep_inspect.py`, default) — execs hardcoded commands into all containers (ps, ss, /proc, df, known config paths + dynamically discovered paths from PID 1 cmdline). Parsing is regex-based via `ai/extractor.py` (no AI). Bumps confidence to 0.7.
3. **Agentic scan phase** (`phases/agentic.py`, `--agentic`) — LLM iteratively decides what commands to exec, interprets results, and emits structured profile updates. Replaces Phase 2 entirely. Bumps confidence to 0.9. Requires `--model` and `--api-key`.
4. **Incremental rescanning** — `knowledge/fingerprint.py` tracks image digests and config versions; skips unchanged workloads unless `--force-rescan`.

### When to use Phase 2 vs Phase 3
- **Phase 2 (default)**: free, deterministic, reproducible, data stays local. Best for regulated environments, CI/CD drift detection, incremental nightly rescans at scale.
- **Phase 3 (`--agentic`)**: ~$0.003/workload with Kimi K2.5. Best for distroless containers, deep dependency discovery, initial cluster exploration. Non-deterministic; sends cluster internals to the LLM provider.
- Do not build a hybrid mode — the two phases are intentionally separate tradeoffs.

### Documentation generation (`kube2docs generate`)
`ai/writer.py` reads scan output, sends structured data to an LLM via `ai/provider.py` (litellm wrapper), produces Markdown docs. `--recommendations` generates separate recommendation files.

### Key modules
- `cli.py` — Click commands (entry point: `kube2docs.cli:main`)
- `scanner.py` — orchestrates phases, wires dependencies
- `knowledge/schemas.py` — Pydantic models for all data structures
- `knowledge/store.py` — JSON file I/O
- `kube/client.py` — wraps official K8s Python client
- `kube/exec.py` — pod command execution with shell detection and timeouts
- `security/hasher.py` — SHA-256 hashing for sensitive values (passwords, tokens, keys)
- `progress/tracker.py` — status JSON + rich terminal output
- `phases/agentic.py` — Phase 3 agentic scan; multi-turn LLM conversation loop with command safety validation (`_is_safe_command`) and artifact storage in `{workload}.raw/agentic/`

## Code Conventions

- Python 3.12+ syntax: `list[str]`, `dict[str, int]`, `X | None` unions
- Line length: 120
- Ruff rules: E, F, I, N, W, UP, B, SIM
- MyPy strict mode
- Every module uses `logger = logging.getLogger(__name__)`
- Pydantic BaseModel for all data schemas; `Literal` for constrained strings; `Field(default_factory=...)` for mutable defaults
- Tests use pytest classes with `tmp_path` fixture for temp directories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matt-szymczyk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
