## harness-engineering

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **one-day workshop course**: "Harness Engineering & Agent Orchestration." It is a
**systems workshop, not an "agent app" workshop.**

**Core idea:** Across the day we build ONE persistent codebase — a **mini agent runtime (the
harness)**: the layer that wraps a dumb LLM loop and makes it production-grade. We do **not**
build a vertical agent. The agent is a deliberately boring, domain-neutral task agent; the
**harness is the protagonist.** Each lesson adds one capability to the runtime.

The mantra repeated all day: **agent systems are workflow systems.** The LLM decides the next
semantic step; the harness owns execution.

## The spine

Everything builds toward one execution loop. Each lesson fills in one line:

```ts
async function runWorkflow(workflowId, input) {
  const state = await stateStore.load(workflowId);
  while (!state.done) {
    const context = await hydrateContext(state);     // L4 memory
    const step    = await agent.next(context);       // the LLM decides
    await policy.check(step);                         // L3 sandbox / L7 approval
    await eventLog.append(workflowId, step);          // L2 durable
    const result  = await executeStep(step);          // tool / handoff / approval
    await stateStore.checkpoint(workflowId, result);  // L2 durable
  }
}
```

## The scenario (domain-neutral, reused every lesson)

One generic **task packet** — review mixed work items, classify, draft actions, pause before
external actions. Tools are fake-but-realistic, split into:

- **Safe** (no approval): `searchKnowledgeBase`, `summarizeDocument`, `classifyItem`, `createDraft`
- **Dangerous** (require approval in L7): `runCode`, `sendMessage`, `updateRecord`, `createTicket`

This single scenario yields routing, memory, supervision, and approval without making the course
about any real product.

## Tech Stack

- **Runtime:** Node.js + TypeScript (run with `tsx`, no build step).
- **LLM provider:** OpenAI via the Vercel AI SDK (default model `gpt-5.5`).
- **Libraries:** Minimal and chosen per lesson (e.g. an isolation primitive in L3, a store in
  L2/L4). Do not lock these in prematurely. We build the concepts from scratch and *name* the
  production tools (Temporal/Inngest, e2b/Firecracker, LangGraph/Mastra) rather than depend on them.
- **Transport:** Express + `ws`. The browser and server speak over ONE WebSocket: client→server
  commands (`submit_task`, later `approve`) and server→client the harness event stream. We do NOT
  use the AI SDK's `useChat`/UI transport — that's request-scoped and can't express server-initiated
  events (a sub-agent finishing, a workflow resuming days later). The AI SDK is used server-side only
  (`streamText`), bridged into our event bus.
- **Inspector UI:** a small, prebuilt web dashboard (Vite + React + prompt-kit/shadcn/Tailwind) in
  `web/`. It renders the harness event stream (step timeline, checkpoints, sub-agent tree, approval
  modal). **Students never build the inspector** — it's purely a projection of the events the harness
  emits, so invisible infra becomes visible (critical for the VOD).
- **Notes site:** VitePress, served from `lessons/`.

Everything runs locally. No deployment.

## Repo structure

```
shared/
  events.ts     AgentEvent union — the contract between harness and UI (grows each lesson)
harness/        the runtime — accretes one module per lesson
  runtime.ts    the spine (runAgent → runWorkflow / start / resume / suspend). STUB in the starter.
  model.ts      the AI SDK OpenAI model (gpt-5.5 default)
  tools.ts      safe + dangerous tool definitions                      — L1+
  state.ts      StateStore (checkpoint/load)                            — L2
  policies.ts   ToolPolicy (beforeToolCall)                            — L3
  memory.ts     Memory + ContextHydrator                               — L4
  router.ts     Router + typed Handoff                                 — L5
  supervisor.ts Supervisor (fan-out/fan-in)                            — L6
  approvals.ts  ApprovalStore + suspend/resume                         — L7
server/
  index.ts      Express + ws; on submit_task calls runAgent with an emit()
  bus.ts        in-memory EventBus + createEmitter (becomes durable in L2)
web/            prebuilt inspector (Vite + React + prompt-kit). NOT taught.
examples/       one runnable scenario per lesson (added as we go)
lessons/        VitePress notes (one folder per lesson)
components.json shadcn config for pulling more prompt-kit components
```

## Commands

```bash
npm run dev         # harness server (:8787) + inspector UI (:5173)
npm run typecheck   # tsc --noEmit across server/shared/harness/web
npm run docs        # serve the lesson notes with VitePress
```

Per-lesson run commands are introduced in each lesson's notes. The starter ships with a STUB
`runAgent` so the app runs end-to-end (browser → ws → server → bus → browser) with the agent loop
left as the hole you fill in Lesson 1.

## Course Structure

Strict rhythm per lesson (this matters most for the VOD): **name the production failure → show
the naive agent fail live → live-code the harness module that fixes it → re-run and watch it
survive → map it to the real production tool.** Lessons are under one hour. 7 lessons in a day,
split Morning (Harness Core: L1–3) / Afternoon (Control Plane: L4–7). See `course.yml`.

### Branching Strategy

Each lesson gets its own branch. The next branch includes the previous lesson's solution so
students who fall behind can catch up by checking out the next branch.

Build forward in one pass: `lesson-1` → ... → `lesson-7` → `main`. Each branch adds **one harness
module** (not a new app). `main`/`complete` point at the latest lesson — by then the harness is
complete. Notes are written alongside the code as we go.

### Lesson Notes

Markdown served as a VitePress site from `lessons/<nn-slug>/index.md`. Each lesson's note uses the
rhythm above: **The pain / The failure mode / What the harness adds / Live code (full diffs) / The
demo / In production.** Add new lessons to the sidebar in `lessons/.vitepress/config.ts`.

## Code Style

This is a live-coding course. All code must be:
- **Simple and readable** — students type this during the workshop.
- **No clever abstractions** — prefer straightforward, explicit code over DRY.
- **Easy to follow** — prioritize clarity over elegance. The failure modes should be impressive,
  not the agent.

## Git Commits

Do not add co-author lines to commits.

---
> Source: [Hendrixer/harness-engineering](https://github.com/Hendrixer/harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
