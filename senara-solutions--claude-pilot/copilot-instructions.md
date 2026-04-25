## claude-pilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**claude-pilot** is a TypeScript CLI that wraps Claude Code using the Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`). It runs Claude Code headlessly and intercepts permission requests and questions via the SDK's `canUseTool` callback, forwarding them to an external agent (e.g., `mika --agent mika-dev ask`) for automated decision-making.

## Commands

```bash
npm run dev -- "prompt"        # Run directly with tsx (development)
npm run build                  # Build with tsup → dist/cli.js
npm start -- "prompt"          # Run built version
npx tsc --noEmit               # Type-check without building
```

Usage: `claude-pilot [options] <prompt>`
- `--task-id <id>` — Task identifier for external agent tracking
- `--no-relay` — Disable agent forwarding, answer all prompts locally
- `--cwd <dir>` — Working directory for Claude Code
- `--verbose` — Show debug output
- `--max-turns <n>` — Maximum agentic turns (default: 200)
- `--max-budget <usd>` — Maximum cost in USD (default: disabled)
- `--stall-threshold <n>` — Consecutive no-tool turns before termination (default: 5)
- `--empty-threshold <n>` — Consecutive trivial responses before termination (default: 5)
- `--idle-timeout <ms>` — Idle timeout in milliseconds (default: 300000)
- `--no-guardrails` — Disable application-level guardrails (stall, empty, idle)

## Architecture

```
src/cli.ts          → Entry point: arg parsing, config loading, signal handling, guardrail CLI flags
src/agent.ts        → SDK query() wrapper, message stream iteration, log rendering, guardrail integration
src/permissions.ts  → canUseTool handler: tier1 filter, relay, retry, interactive fallback, idle timer pause/resume
src/tier1.ts        → Tier 1 auto-approval filter: deny-list, safe command patterns, path safety
src/transport.ts    → execFile command transport with Zod validation
src/guardrails.ts   → Session termination guardrails: stall detection, empty-response, idle timeout
src/ui.ts           → Stderr log renderer (ANSI colors), guardrail log functions
src/types.ts        → PilotEvent, PilotResponse (Zod schema), PilotConfig, GuardrailConfig, ResultJson
```

**Flow**: CLI → `query()` with `canUseTool` callback → on tool permission needed → format `PilotEvent` → invoke external command via `execFile` (stdin JSON) → validate response with Zod → map to SDK `PermissionResult` → return to SDK.

**Key design decisions**:
- External agent is a **black box** — claude-pilot sends events and waits. If the agent escalates internally (e.g., asks a human via Telegram), it just takes longer to respond. claude-pilot is unaware.
- Response contract is minimal: `{action: "allow"}`, `{action: "deny"}`, `{action: "answer", answers: {...}}`
- Malformed JSON from the agent triggers one retry with error feedback, then falls back to interactive user prompt
- Sub-agent tool calls are auto-allowed (not forwarded)
- Non-interactive mode (no TTY) auto-denies on failure
- `execFile` (not `exec`) prevents shell injection
- Sensitive env vars (`ANTHROPIC_API_KEY`, etc.) are scrubbed before spawning commands
- **Session guardrails** detect degenerate loops (stall, empty responses, idle) and terminate via `AbortController` with structured `ResultJson` output. SDK-native `maxTurns`/`maxBudgetUsd` are passed through to `query()`. Idle timer pauses during `canUseTool` to avoid false positives from slow relay agents.
- **Pipeline slash commands bypass mika-dev approval.** The `Skill` tool invocations for `/mika`, `/ce:*`, `/ralph-loop:*`, `/compound-engineering:resolve_todo_parallel`, and `/mika-doc-audit` are auto-approved at TIER 1. These are the agent's own orchestration steps — routing them through the relay exposes them to LLM-driven approval that can rationalize fabricated denials. The allowlist is in `TIER1_SAFE_SKILLS` in `src/tier1.ts`.

## Environment Variables

Place a `.env` file in the claude-pilot root directory (alongside `package.json`) to set process-level env vars. Values do NOT override existing `process.env` entries. See `.env.example` for available variables.

Example: create `.env` with `GH_TOKEN=<bot-pat>` so PRs appear from the bot account while agents use the host's `gh auth` for `gh pr checks`.

The `.env` file is gitignored and not copied to worktrees. Autonomous sessions inherit env vars from the parent process instead.

**Note:** Variables matching sensitive patterns (`TOKEN`, `KEY`, `SECRET`, `AUTH`, etc.) are scrubbed from the relay child process via `scrubEnv()` in `transport.ts`, but remain visible to the Claude Code SDK subprocess. This is by design — Claude Code needs tokens like `GH_TOKEN` to operate, while the relay agent should not see them.

## Configuration

Place `claude-pilot.json` in the target project's `.claude/` directory:
```json
// .claude/claude-pilot.json
{
  "command": "mika",
  "args": ["--agent", "mika-dev", "ask"],
  "timeout": 120000,
  "guardrails": {
    "maxTurns": 200,
    "stallThreshold": 5,
    "emptyResponseThreshold": 5,
    "idleTimeoutMs": 300000,
    "minTurnsBeforeDetection": 10
  }
}
```

Guardrail fields are optional — defaults apply when omitted. CLI flags override config file values. Set a threshold to `0` to disable that specific guardrail.

## Key SDK Types

- `canUseTool(toolName, input, options)` — options includes `signal` (AbortSignal), `agentID` (sub-agent), `toolUseID`, `decisionReason`, `blockedPath`
- `PermissionResult` — `{behavior: "allow", updatedInput}` or `{behavior: "deny", message}`
- `AskUserQuestion` — intercepted via `canUseTool` when `toolName === "AskUserQuestion"`, response requires `{behavior: "allow", updatedInput: {questions, answers}}`

## Planning Documents

- Brainstorm: `docs/brainstorms/2026-03-15-claude-pilot-brainstorm.md`
- Plan: `docs/plans/2026-03-15-001-feat-claude-pilot-sdk-wrapper-plan.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/senara-solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
