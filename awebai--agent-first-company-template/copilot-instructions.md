## agent-first-company-template

> This file is read once when setting up the team. It explains the

# AGENTS.md — how agents work in this team

This file is read once when setting up the team. It explains the
shape; the day-to-day operating instructions are in `CLAUDE.md` (top-
level) and per-agent `agents/<name>/CLAUDE.md` files.

## Each agent is a named persona with persistent context

Not just a chat session. An agent has:

- A **name** (e.g., `lead`, `builder`, `runner` — or whatever your work
  surfaces map to)
- A **responsibility area** clearly bounded from the others
- A **persistent context** that survives any single session
  (`agents/<name>/CLAUDE.md` + `agents/<name>/handoff.md` carry that
  across resets)
- A **way to receive direction** (from humans or other agents)
- A **way to communicate results** (status updates, handoffs, optionally
  direct messages to other agents)

## Roles included in this template

Three example roles to start. Rename, add, or remove based on your
actual work surfaces.

### `lead`

Direction: priorities, decisions, framing. The role that holds "what
matters now."

### `builder`

The work itself. Whatever your company builds — code, content,
designs, sales conversations.

### `runner`

Ops: ship, verify, dashboards. The role that closes the loop between
build and visible-live.

## When to add an agent

Add an agent when the work needs a new surface. Examples of when to
add:

- The work has a distinct rhythm or cadence the existing surfaces
  don't carry well (e.g., customer support is reactive; the builder
  surface is proactive — they don't share a clock)
- Two existing surfaces keep stepping on each other's work (the
  collision IS the signal you need a third surface)
- A specific customer-facing or external-facing role needs its own
  voice (e.g., outreach, customer success)

Don't add agents preemptively. The shape is downstream of the
principles.

## When to remove an agent

If a surface is rarely-active and its work is naturally absorbed by
another surface, fold it in. Don't keep ghost agents.

## How agents communicate

### Stage 1-2 (no cross-AI messaging)

Humans relay between agent personas. "Builder, here's what Lead just
decided about X." This works for small teams.

### Stage 3+ (with cross-AI messaging via `aw` or similar)

Agents have addresses. They can mail (async) or chat (sync) each other
directly. The human reads the result rather than relaying.

This template includes scaffolding for the `aw`-CLI-based setup (the
project that builds the agent-coordination layer we use). For more on
that, see [github.com/awebai/aweb](https://github.com/awebai/aweb).

## How decisions get made

Each agent owns its surface; the outcome belongs to all of them. When
peers see something differently, they work it out together. When they
genuinely can't converge, the human founder decides — but that
escalation should be rare and worth using.

When the plan changes, whoever changed it adds an entry to
`docs/decisions.md`.

## How knowledge persists

Three places:

1. **Status files** (`status/<surface>.md`) — current state of each
   surface; updated on wake-up
2. **Decision log** (`docs/decisions.md`) — append-only record of plan
   changes
3. **Handoff documents** (`agents/<name>/handoff.md`) — per-agent
   in-flight context for the next session

A fresh agent (or a human looking at the repo cold) should be able to
inspect these and understand what's happening.

---
> Source: [awebai/agent-first-company-template](https://github.com/awebai/agent-first-company-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
