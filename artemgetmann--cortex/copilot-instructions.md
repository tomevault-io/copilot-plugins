## cortex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Cortex?

Cortex is an AI agent project focused on persistent memory across sessions. The active track in this branch is the CLI Memory V2 lab (`tracks/cli_sqlite/`), where the agent learns from failures and recovers faster over repeated runs.

The original FL Studio computer-use track is preserved as historical context under `docs/archive/fl-studio-legacy/`.

**Hackathon project:** Anthropic "Built with Opus 4.6" (Feb 10-16, 2026).

## Project North Star (Important)

The goal is **not** to build the best FL Studio autopilot.

The goal is to test whether lessons improve agent performance over repeated attempts and whether those gains generalize across domains/tasks. FL Studio is one stress-test environment for computer-use, not the product end-state.

When evaluating progress, prioritize:
- Does the agent avoid repeating the same mistakes after failures?
- Do post-task lessons/patches get applied and produce measurable lift?
- Does learning transfer beyond one UI/task shape (especially to CLI Memory V2 domains)?

If FL benchmarks are unstable due to environment issues (lock screen, hidden window, capture failures), treat those runs as infra-invalid and do not use them as evidence against learning quality.

## Commands

```bash
# Install dependencies (Python 3.11+, macOS only)
pip install -r requirements.txt

# Configure
cp .env.example .env   # then set OPENAI_API_KEY

# Run an agent session
python3 scripts/run_agent.py \
  --task "Create a 4-on-the-floor kick drum pattern" \
  --session 2201 --max-steps 80 --verbose

# Run without skills (baseline comparison)
python3 scripts/run_agent.py --task "..." --session 1 --no-skills

# Override model
python3 scripts/run_agent.py --task "..." --model claude-haiku-4-5

# Use subscription-backed Claude CLI transport (no API key needed for executor loop)
python3 scripts/run_agent.py --task "..." --llm-backend claude_print
```

There is no formal lint/build pipeline yet. Verify runtime behavior with agent sessions and check `sessions/session-NNN/` artifacts (`events.jsonl`, `metrics.json`, `step-NNN.png`).

Test suites:

```bash
# Root FL/runtime smoke tests
python3 -m pytest tests -q

# Active Memory V2 CLI track tests
python3 -m pytest tracks/cli_sqlite/tests -q
```

## API Limit Policy (Execution Priority)

- Default benchmark/runtime path is `--llm-backend openai` (API) for speed and cost control.
- If API quota/rate limits are hit during active collaboration, stop and notify the user immediately.
- Do not silently switch to `claude_print`; it is typically much slower for benchmark iteration.
- Only use `claude_print` when the user explicitly asks for it, or for unattended overnight runs.

## Architecture

```
agent.py          ← FL Studio loop orchestrator
computer_use.py   ← macOS Quartz CGEvent wrapper (key, click, screenshot, coordinate mapping)
config.py         ← Env-based config loader (CortexConfig dataclass)
memory.py         ← Session path management + JSONL/metrics I/O
consolidate.py    ← Post-session skill generation from logs (stub, not yet implemented)
claude_print_runtime.py ← Shared Claude Print backend helpers (JSON parsing, model/effort/env resolution)
claude_print_client.py  ← Anthropic-compatible client shim over `claude -p`

scripts/
  run_agent.py    ← CLI entry point (argparse → run_agent())

skills/fl-studio/ ← Markdown skill docs loaded into context
  basics/SKILL.md
  drum-pattern/SKILL.md
  drum-pattern/CONTRACT.json

sessions/         ← Per-session output (gitignored)
  session-NNN/    ← events.jsonl + metrics.json + step-NNN.png screenshots

docs/
  README.md                         ← Canonical docs index (active vs archive)
  MEMORY-V2-EXECUTION-PLAN.md      ← Living execution plan
  MEMORY-V2-AGNOSTIC-PLAN.md       ← Memory V2 requirements + status
  MEMORY-V2-BENCHMARKS.md          ← Benchmark runbook
  MEMORY-V2-CURRENT-FLOW.html      ← Current runtime diagram
  archive/                          ← Historical docs only (not source of truth)
```

### Data flow

1. `run_agent()` builds system prompt + loads skills from `skills/fl-studio/` into context
2. Sends task to OpenAI API (`llm-backend=openai`) or Anthropic API/Claude CLI when explicitly selected
3. Model returns `tool_use` blocks (screenshot, key, click, etc.)
4. `ComputerTool.run()` executes via macOS Quartz CGEvent APIs, returns screenshot
5. Loop continues until model stops requesting tools or hits `max_steps`
6. Events logged to JSONL, metrics written to JSON, screenshots saved as PNGs

### Key design decisions

- **No database/vector store.** Opus 4.6 has 1M token context — skills loaded directly into prompt.
- **Prompt caching** on system blocks + recent user turns (~80% cost reduction on repeated context).
- **Quartz CGEvent APIs** (not pyautogui) for reliable macOS input delivery.
- **Bundle ID matching** (`com.image-line.flstudio`) to find FL Studio, not window title.
- **Coordinate mapping:** API operates in 1024x768 space, mapped to FL Studio window bounds at runtime.
- **UI settle detection:** Post-action screenshot polling with image similarity threshold prevents race conditions.

## macOS / Quartz gotchas

- FL Studio **must be visible and forefront** for input delivery to work.
- `CGEventPostToPid` requires Accessibility permissions granted to the terminal running the script.
- Claude Code's sandbox blocks Quartz/CGEvent APIs silently — use `dangerouslyDisableSandbox: true` for any Bash commands that invoke Quartz (screenshots, key events, window queries).
- `CGWarpMouseCursorPosition` works even with sandbox (different API path).
- `computer_use.py` forbids dangerous key combos (cmd+q, cmd+tab, cmd+w, cmd+m).

## Computer Tool Compatibility (Important)

- Decider models (default Haiku/Sonnet path) use `computer_20250124`.
- Heavy model (default Opus path) uses `computer_20251124`.
- The `zoom` action is available only with `computer_20251124` (Opus path).
- Do not ask Haiku/Sonnet runs to use zoom; they should use screenshot + precise clicks instead.
- If you need zoom-dependent precision checks, run with Opus.

## CLI Learning Lab (`tracks/cli_sqlite/`)

The CLI lab is the active Memory V2 harness. It tests whether an LLM agent can learn across domains/tasks (`gridtool`, `fluxtool`, `sqlite`, `shell`, `artic`) through error capture + lesson retrieval/promotion.

### Running experiments

**Always run these in background** — they take 3-10 minutes and make API calls:

```bash
# Single session
python3 tracks/cli_sqlite/scripts/run_cli_agent.py \
  --task-id aggregate_report --domain gridtool \
  --session 9501 --max-steps 6 --bootstrap --verbose

# Learning curve experiment (10 sequential sessions)
python3 tracks/cli_sqlite/scripts/run_learning_curve.py \
  --task-id aggregate_report --domain gridtool \
  --sessions 10 --start-session 9501 --max-steps 6 \
  --bootstrap --verbose --posttask-mode direct
```

Key flags:
- `--bootstrap`: No skill docs, agent learns from lessons + error messages only
- `--cryptic-errors`: Strip helpful hints from error messages (harder mode)
- `--max-steps N`: Step budget (6 = tight, 12 = generous)
- `--posttask-mode direct`: Apply skill patches immediately (vs `candidate` for queuing)

Core test command:
```bash
python3 -m pytest tracks/cli_sqlite/tests -q
```

Before re-running experiments, clear lessons for a clean baseline:
```bash
cp tracks/cli_sqlite/learning/lessons.jsonl tracks/cli_sqlite/learning/lessons.jsonl.bak
: > tracks/cli_sqlite/learning/lessons.jsonl
```

### API limit handling policy

- If OpenAI or Anthropic API limit/quota is hit, stop immediately and notify the user to raise limits.
- Do **not** continue with `claude_print` fallback by default because it is much slower and wastes iteration time.
- Only use `claude_print` after an API limit hit if the user explicitly says to continue unattended (example: user says they are going to sleep and wants overnight progress).

## OpenClaw AGI Bridge (Isolated Runtime)

Status: deprecated path for Telegram serving. Keep only as legacy bridge docs.

The OpenClaw AGI bot must run in an isolated profile so the existing `~/.openclaw` bot is never affected.

Rules:
- Existing bot profile: `~/.openclaw` (do not modify for AGI bridge work).
- Legacy AGI OpenClaw profile backups moved to:
  `integrations/legacy/openclaw-profiles/` (gitignored, local-only).
- Do not use legacy OpenClaw AGI profiles for live Telegram serving.
- AGI workspace (inside Cortex for visibility): `integrations/openclaw-agi/workspace`.
- Bridge script: `integrations/openclaw-agi/workspace/bin/cortex_cli_bridge.sh`.
- Dispatcher script (chat/task router): `integrations/openclaw-agi/workspace/bin/cortex_openclaw_dispatch.sh`.

Why:
- Cortex remains the single source of truth for learning logic.
- OpenClaw stays a thin Telegram ingress/egress connector.
- Any improvement in `tracks/cli_sqlite` immediately improves AGI bot behavior.

Setup and run:
```bash
cd /Users/user/Programming_Projects/Cortex
./scripts/openclaw_agi_setup.sh
./scripts/openclaw_agi_start.sh
```

Enable live Telegram testing with a dedicated token:
```bash
OPENCLAW_AGI_TELEGRAM_BOT_TOKEN="YOUR_NEW_BOT_TOKEN" ./scripts/openclaw_agi_setup.sh
```

Task-mode protocol (for live chat):
- `/run ...` => execute Cortex learning loop and persist lessons.
- `/run ... learn=off` => execute task without writing lessons (safe live smoke test).
- `/learn-status` => summarize recent learning signals.
- Anything else => chat mode (no lesson writes).

## Standalone Telegram AGI Frontend (Preferred for Fast Iteration)

Source of truth for Cortex Telegram bot runtime:
- Bot code: `integrations/cortex-telegram-agi-bot`
- LaunchAgent label: `com.cortex-telegram-agi`
- Startup script: `scripts/cortex_tg_agi_start.sh`
- Install script: `scripts/cortex_tg_agi_install_launchagent.sh`
- Bot username target: `@cortex_openclaw_agi_bot`

Safety rule:
- Do not run `@cortex_openclaw_agi_bot` through OpenClaw gateway profiles.
- Keep Telegram disabled in legacy AGI OpenClaw profiles unless explicitly doing legacy bridge debugging.

For live testing without touching OpenClaw or the existing `claude-code-telegram-bot` repo, use:

- `integrations/cortex-telegram-agi-bot`
- `scripts/cortex_tg_agi_start.sh`
- `scripts/cortex_tg_agi_install_launchagent.sh`
- `scripts/cortex_tg_agi_uninstall_launchagent.sh`

Worktree/runtime rule (important):
- LaunchAgent is pinned to the checkout where you run install.
- If you switch to another worktree branch, re-run install there so live Telegram serves that checkout:
  - `./scripts/cortex_tg_agi_install_launchagent.sh`
- Startup now supports dynamic path binding by default (`CORTEX_DYNAMIC_PATHS=1`):
  - `CORTEX_ROOT`, `CORTEX_DISPATCHER_PATH`, and `AI_WORKING_DIR` auto-bind to the current checkout.
  - Set `CORTEX_DYNAMIC_PATHS=0` only if you intentionally want static pinned paths in `.env`.
- Quick verification:
  - `launchctl list | rg com.cortex-telegram-agi`
  - `PID=$(launchctl list | awk '/com.cortex-telegram-agi/{print $1}')`
  - `lsof -a -d cwd -p "$PID" | tail -n +2`

Design:
- Telegram bot is frontend only.
- Cortex (`tracks/cli_sqlite`) is the brain.
- Task-mode is routed to Cortex via dispatcher:
  - `integrations/openclaw_agi_dispatch.py`

Modes:
- `/run ...` => Cortex learning loop.
- `/learnstatus` (or `/learn-status`) => learning metrics summary.
- normal chat => regular assistant response.
- optional auto-detect asks confirmation before routing task-like messages.

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | (required for `--llm-backend openai`) | API key |
| `ANTHROPIC_API_KEY` | (required for `--llm-backend anthropic`) | API key |
| `CORTEX_MODEL_HEAVY` | `claude-opus-4-6` | Main agent model |
| `CORTEX_MODEL_DECIDER` | `claude-haiku-4-5` | Cheaper model for gate tests |
| `CORTEX_DISPLAY_WIDTH_PX` | `1024` | API coordinate space width |
| `CORTEX_DISPLAY_HEIGHT_PX` | `768` | API coordinate space height |
| `CORTEX_ENABLE_PROMPT_CACHING` | `1` | Enable prompt caching |
| `CORTEX_CLAUDE_PRINT_MODEL` | `claude-opus-4-6` | Model used when `--llm-backend claude_print` |
| `CORTEX_CLAUDE_PRINT_EFFORT` | `high` | Effort level for `claude -p` (`low`/`medium`/`high`) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artemgetmann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
