## agent-team-manager

> Offload any non-trivial task to a background agent by default so the main agent stays free; persist named domain experts on request


# Agent Team Manager

**Default posture: offload, don't block.** If a task would occupy you (the main
agent) for more than a few seconds, hand it to a background agent and stay
available for the next request. Only trivial, near-instant work is handled inline.

## The decision rubric — apply to EVERY task

**Handle inline** only if ALL of these are true:
- answerable in **0–1 tool calls**
- finishes in **~a few seconds**
- no file edits, no repeated external/shell calls, not multi-step

Examples: a fact or definition, a unit conversion, a single quick file read, a
one-line edit, "what time is it".

**Offload to a background agent** if ANY of these are true:
- likely needs **≥2 tool calls** or **>~10 seconds**
- edits files, hits the network/shell repeatedly, or is multi-step/ongoing

Examples: research, code changes across files, builds/tests, data analysis,
drafting documents, anything investigative or open-ended.

**When unsure → offload.** Bias toward keeping yourself free. The whole point is
that a 30-second task must not block the user's next request.

## How to offload

1. Decide the domain (research, code, design, writing, analytics, market,
   customer, or general).
2. Spawn it as a **background task** using your native background-task / subagent
   tool, with **notify policy = completion + errors** so results and failures are
   pushed back to the user automatically.
3. The skill's helper records the task. Invoke:
   `node agent-team-manager.js task "<the task>"`
   This spawns the background worker, logs it, and tracks domain recurrence.
4. Immediately tell the user you've offloaded it and are free for the next
   request. Do **not** wait for the background task inline.

## Hybrid persona model

- **Default = ephemeral worker.** One-off tasks get a fire-and-forget background
  worker. No persistent persona, nothing to manage.
- **Persist on request.** When the user says "make a dedicated agent for X" (or
  similar), create/reuse a named domain expert:
  `node agent-team-manager.js task --persist "<the task>"`
- **Promote on recurrence.** On the user's **3rd** task in the same domain, the
  helper surfaces an offer to promote it to a standing named agent. Pass the offer
  along; only create the persistent agent if the user accepts.

## Notifications

All background tasks notify on **completion and errors**. Completed results are
delivered back to the user automatically; failures surface proactively. You never
need to poll inline.

## Commands

```
node agent-team-manager.js task "<task>"            # offload (default, ephemeral)
node agent-team-manager.js task --persist "<task>"  # offload to a persistent named agent
node agent-team-manager.js tasks                    # list background tasks + status
node agent-team-manager.js done <id>                # mark a background task completed (keeps the log + prune accurate)
node agent-team-manager.js fail <id>                # mark a background task failed
node agent-team-manager.js list                     # list persistent named agents
node agent-team-manager.js query <agent> "<q>"      # ask a persistent agent
```

## Tracking completion

OpenClaw's background-task runtime notifies on completion and errors. When a
background task finishes, mark it in the skill's log so `tasks` listings and the
7-day auto-prune stay accurate:

- `node agent-team-manager.js done <id>`  (completed)
- `node agent-team-manager.js fail <id>`  (failed)

## State

- `agents.json` — persistent named agents only (name, domain, status, lastUsed,
  taskCount) plus per-domain recurrence counts.
- `tasks.json` — ephemeral background-task log (auto-pruned after 7 days).

## Security

- Background agents run in isolated sessions; no agent reads another's context.
- All spawns are logged in `tasks.json`.

---
> Source: [toorcn/agent-team-manager](https://github.com/toorcn/agent-team-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
