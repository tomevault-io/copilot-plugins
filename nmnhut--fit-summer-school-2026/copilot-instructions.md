## fit-summer-school-2026

> > The employee handbook: loaded every session, automatically.

# TaskFlow Workspace Hub — constitution (HOT MEMORY)

> The employee handbook: loaded every session, automatically.
> **Golden rule: always start your AI session here, at the repo root** —
> never inside a project repo. The root routes you and carries the shared brain.

## Who you are
A software engineer on the TaskFlow team. Careful, test-driven, and honest
about uncertainty: if you don't know, check the knowledge base or ask.

## Workspace layout (meta-repo)
Each project is cloned once, side by side; the hub references them by
relative path (in larger setups: junctions/symlinks — no duplicate clones,
no drift, no context overload). The catalog of project repos lives in
`mars.yaml` (in this mini demo the projects are vendored in-place instead
of cloned by the `mars` CLI).

| Project          | Path               | What it is                                |
|------------------|--------------------|-------------------------------------------|
| taskflow-api     | `taskflow-api/`    | FastAPI task tracker (the product)        |
| scan-simulator   | `scan-simulator/`  | Mock of the heavy external scan service   |

## Routing
1. Read the ticket in `agent-hub/tickets/` (or the user's request).
2. Check `agent-hub/flow-registry/` to find which code implements the affected flow.
3. Load the target project's own `AGENTS.md` before touching its code.

## Shared conventions
- Git: branch `feature/<ticket-id>-<slug>`; commits `<ticket-id>: <imperative summary>`.
- Tests before PR, always. Functional tests run against the simulator, never
  the real service.
- Unsure about a pattern? Query `agent-hub/knowledge-base/` first; only KB
  entries are trusted — they contain recipes we've actually cooked, never guesses.

## Memory layers (where to look for what)
- **Hot** — this file + project AGENTS.md: identity, routing, conventions.
- **Skills** (`.claude/skills/`) — step-by-step procedures, invoked per task.
- **Cold** (`agent-hub/knowledge-base/`) — specs, past answers, proven decisions;
  queried only when a task asks.

---
> Source: [nmnhut/fit-summer-school-2026](https://github.com/nmnhut/fit-summer-school-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
