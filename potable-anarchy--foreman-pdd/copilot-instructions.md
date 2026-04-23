## foreman-pdd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **PDD Foreman Agent** - a hackathon project for SF Tech Week's Agent Builders Hackathon that orchestrates the generation of working software from Product Design Documents (PDDs). The agent reads `.prompt` or `.md` design specs and uses the `pdd` CLI to generate, test, and fix projects while coordinating multiple sponsor integrations.

The meta-builder approach: text spec → orchestration → working software with observability, voice narration, and visual diagrams.

## Key Commands

### PDD CLI Operations

The `pdd` CLI is central to this project. **Important**: Always set `PDD_PATH` before running PDD commands to avoid "could not determine project root" errors:

```bash
export PDD_PATH="$PWD"
```

**Primary build workflow:**
```bash
# Generate and sync from a prompt file (basename only, not path)
pdd --local --force sync foreman_agent --target-coverage 80

# Fix failing tests, then re-sync
pdd --local fix foreman_agent
pdd --local --force sync foreman_agent --target-coverage 80
```

**Common wrapper (from agents.md):**
```bash
# Convenience function for consistent runs
foreman() {
  export PDD_PATH="${PDD_PATH:-$PWD}"
  local base="${1:-foreman_agent}"
  shift || true
  pdd --local --force sync "$base" --target-coverage 80 "$@"
}

# Usage:
foreman                    # builds prompts/foreman_agent.prompt
foreman demo_todo          # builds prompts/demo_todo.prompt
```

### Development Commands

Once the project is implemented, expect these commands:

```bash
# Install dependencies
pnpm i --prefer-offline

# Run tests (from root or specific workspace)
pnpm -r test              # run all tests across workspaces
pnpm test                 # run tests in current package

# Development servers
pnpm -r dev               # start all dev servers
cd apps/web && pnpm dev   # Next.js dashboard
cd apps/api && npx convex dev  # Convex backend
```

## Architecture

### Core Orchestration Flow

```
Spec Input (.prompt/.md)
  → Dedalus Agent (dedalus/agent.yaml)
    → Foreman Loop (packages/foreman/loop.ts)
      → PDD CLI (generate → sync → fix on failure)
      → Parallel Integrations:
         - Convex: Log runs/tasks/state
         - Vibe Kanban: Update task board
         - ElevenLabs: Narrate milestones
         - Fireworks AI: Generate architecture diagram
         - OpenAI: Normalize specs & reason about failures
  → Working Project + Dashboard
```

### Key Components

**packages/foreman/**
- `cli.ts`: CLI entrypoint for triggering builds
- `loop.ts`: Main orchestration logic - calls PDD and coordinates integrations
- `tools/pdd.ts`: Spawns PDD CLI and collects output
- `tools/voice.ts`: ElevenLabs TTS integration for milestone narration
- `tools/kanban.ts`: Vibe Kanban API integration (or local stub)
- `tools/diagram.ts`: Fireworks AI diagram generation

**apps/api/convex/**
- `schema.ts`: Defines `runs`, `tasks`, `specs` tables
- `runs.ts`: Mutations/queries for logging build progress
- Persistence layer for all build state and history

**apps/web/**
- `pages/index.tsx`: Dashboard showing run logs, Kanban board, and diagram
- `components/RunList.tsx`: Display Convex-stored runs

**dedalus/agent.yaml**
- Dedalus agent manifest defining tools, model config, and triggers
- Enables HTTP webhook or manual triggering of builds

### Prompt Files

Located in `prompts/`:
- `foreman_agent.prompt`: Design spec for the Foreman Agent itself (meta!)
- `demo_todo.prompt`: Minimal demo app spec for live presentations

**Critical**: PDD commands take the **basename** (e.g., `foreman_agent`) NOT the full path (NOT `prompts/foreman_agent.prompt`).

## Integration Patterns

### Convex Schema Pattern

```typescript
// Expected schema structure (apps/api/convex/schema.ts)
runs: {
  specName: string,
  startedAt: number,
  endedAt: number,
  status: "planning" | "building" | "testing" | "done" | "error",
  summary: string,
}

tasks: {
  runId: string,
  title: string,
  state: "backlog" | "doing" | "done" | "error",
  diffSummary: string,
}

specs: {
  name: string,
  raw: string,
  normalized: string,  // OpenAI-processed version
}
```

### Retry Logic

The agent implements retry logic (max 3 attempts) for failed builds:
1. Run `pdd sync`
2. If tests fail → `pdd fix` → retry `pdd sync`
3. Log each attempt to Convex
4. Update Kanban and voice narration on each state change

### Voice Milestones

ElevenLabs announces key events:
- "Scaffold complete."
- "Tests failed; attempting repair."
- "All tests green. Build complete."

Gracefully degrades to console logs if API keys are missing.

## Environment Setup

Required API keys (set in `.env` or provider vault):

```bash
OPENAI_API_KEY=sk-...
ELEVENLABS_API_KEY=...
FIREWORKS_API_KEY=...           # optional for diagram generation

VIBE_KANBAN_BASE_URL=...
VIBE_KANBAN_TOKEN=...

CONVEX_DEPLOYMENT=              # e.g., prod, dev
CONVEX_URL=                     # if needed by client
```

## Prerequisites

- macOS/Linux with Node 18+
- `uv` (recommended) or Python + pip for PDD CLI
- PDD CLI installed: `uv tool install pdd-cli`
- Configure PDD: `pdd setup` (walk through model/keys setup)

## Common Issues

**"Could not determine project root"**
→ Always export `PDD_PATH="$PWD"` before running PDD commands

**"No such option: --budget/--max-cost"**
→ Some PDD versions don't expose these flags; omit them and use only `--target-coverage`

**"Sync can't find the prompt"**
→ Use basename only: `pdd sync foreman_agent` NOT `pdd sync prompts/foreman_agent.prompt`

**Tests keep failing after multiple fix attempts**
→ Check logs in Convex for error patterns; manually inspect generated test files; consider adjusting the prompt spec for clarity

**Integration APIs unavailable**
→ All integrations (Vibe, ElevenLabs, Fireworks) are designed to be optional; core functionality (PDD + Convex) works standalone

## Testing Strategy

Target: 80% coverage

Expected test files:
- `tests/cli.test.ts`: CLI parsing and PDD command execution
- `tests/convex.test.ts`: Convex logging correctness
- `tests/voice.test.ts`: ElevenLabs API mocking
- `tests/kanban.test.ts`: Vibe Kanban API calls
- `tests/pdd_loop.test.ts`: Mock PDD execution and retry logic

## Demo Flow (3-4 minutes)

1. Show a spec file (e.g., `prompts/demo_todo.prompt`)
2. Run: `foreman demo_todo`
3. Display dashboard with:
   - Planning → Building → Testing status
   - Vibe Kanban board updating in real-time
   - Voice narration of milestones
4. Launch the generated app to show it works
5. Display Fireworks AI-generated system diagram
6. Show Convex run logs with full history

## Hackathon Context

Built for the SF Tech Week $33K Agent Builders Hackathon.

**Sponsor integrations:**
- Dedalus Labs: Agent runtime/orchestration
- Convex: Backend state and persistence
- OpenAI: Spec normalization and reasoning
- ElevenLabs: Voice narration
- Fireworks AI: Visual diagram generation
- Windsurf/Devin: Build workspace environment
- Vibe Kanban: Task visualization

**Judge talking points:**
- Meta-builder: Spec → working software automatically
- Multi-modal output: Voice + visuals + web UI
- Full observability: Every run logged and queryable
- Sponsor integration showcase
- Time-to-value: Minutes from spec to running app

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/potable-anarchy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
