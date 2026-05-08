## tonesoul52

> Purpose: give lower-context GitHub/Copilot style agents a clean entry path into ToneSoul without forcing a repo-wide guess.

# ToneSoul Copilot Instructions

Purpose: give lower-context GitHub/Copilot style agents a clean entry path into ToneSoul without forcing a repo-wide guess.
Status: active repo-local AI entrypoint
Last Updated: 2026-04-02

## Start Here

Read in this order:

1. `AI_ONBOARDING.md`
2. `docs/AI_QUICKSTART.md`
3. `DESIGN.md`

Then run:

```bash
python scripts/start_agent_session.py --agent <your-id> --no-ack
python scripts/run_observer_window.py --agent <your-id>
```

Only after that, if needed:

```bash
python scripts/run_r_memory_packet.py --agent <your-id>
python -m tonesoul.diagnose --agent <your-id>
```

Do not start by scanning the whole repository.

## Current Honest Posture

- ToneSoul is currently `guided collaborator beta`.
- It is not `public_launch_ready`.
- Launch-default coordination is `file-backed`.
- Observer window, subject snapshot, working style, and compaction carry-forward are advisory surfaces.
- Council confidence is currently `descriptive_only`; agreement does not equal calibrated accuracy.

## What To Preserve

Keep these separations visible:

- canonical truth vs runtime posture
- advisory continuity vs durable identity
- descriptive metrics vs calibrated accuracy
- ack vs apply vs promote

If a surface is not clearly canonical, do not silently promote it.

## Default Commands

```bash
python scripts/start_agent_session.py --agent <your-id> --no-ack
python scripts/run_observer_window.py --agent <your-id>
python scripts/run_r_memory_packet.py --agent <your-id>
python -m tonesoul.diagnose --agent <your-id>
python scripts/run_collaborator_beta_preflight.py --agent <your-id>
```

Close out with:

```bash
python scripts/end_agent_session.py --agent <your-id> --summary "<short summary>" --path "<path>"
```

## Working Rules

- Prefer bounded helper extraction over new conceptual lanes.
- Prefer readout parity and regression coverage over new prose-only contracts.
- Park external inspirations under `docs/plans/` before touching `task.md`.
- Do not let a newer planning note silently outrank code, tests, or canonical architecture files.
- If you fail three times on the same path, stop and reassess instead of widening scope.

## Protected / Human-Managed Surfaces

Do not casually edit:

- `AGENTS.md`
- `CLAUDE.md`
- `memory/autonomous/session_traces.jsonl`
- `memory/autonomous/zone_registry.json`
- `OpenClaw-Memory`
- `.claude/`

## Current Successor Goal

The current design pressure is:

`make hot-memory, observer-window, and successor collaboration easier to understand and continue without hidden chat history`

That means:

- later agents should reconstruct the current center from bounded surfaces
- handoff should survive model changes without identity inflation
- summaries must not outrank their parents
- no fake completion on weak evidence

## When In Doubt

Return to:

- `DESIGN.md`
- `docs/status/codex_handoff_2026-04-02.md`
- `docs/plans/tonesoul_successor_collaboration_and_hot_memory_program_2026-04-02.md`

If a claim sounds smoother than its evidence, lower the claim instead of raising the rhetoric.

---
> Source: [Fan1234-1/tonesoul52](https://github.com/Fan1234-1/tonesoul52) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
