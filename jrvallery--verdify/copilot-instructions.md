## verdify

> This repo is worked by several Claude agents in parallel plus a human coordinator (Jason). Every session that edits code here should read this file first.

# Verdify — Agent Working Guide

This repo is worked by several Claude agents in parallel plus a human coordinator (Jason). Every session that edits code here should read this file first.

## What Verdify is

An AI-driven climate controller for a single 367 sq ft greenhouse in Longmont, CO. **Production** — plants are alive, the ESP32 is in the loop every 5 s, the planner runs on real data. Keeping the greenhouse operational ("Track A") always outranks SaaS/cloud refactor progress ("Track B"). See `README.md` for the architecture one-pager.

## Agents

Five persistent agents, each owning one scope. Branches are prefixed by agent name; worktrees live at `/mnt/iris/verdify-worktrees/{agent}/`. Per-agent scope docs live in `docs/agents/`.

| Agent | Owns | Branch prefix | Scope doc |
|---|---|---|---|
| [`firmware`](docs/agents/firmware.md) | ESP32 C++ (`greenhouse_logic.h`), ESPHome YAML, firmware replay, OTA, sensor health | `firmware/*` | `docs/agents/firmware.md` |
| [`ingestor`](docs/agents/ingestor.md) | `ingestor/*.py`, setpoint dispatcher, HA/Shelly/Tempest sync, `alert_monitor`, daily snapshot | `ingestor/*` | `docs/agents/ingestor.md` |
| [`genai`](docs/agents/genai.md) | `iris_planner.py`, `mcp/server.py`, `templates/`, prompts, scorecard/lessons/plan-evaluation | `genai/*` | `docs/agents/genai.md` |
| [`web`](docs/agents/web.md) | `api/main.py`, `scripts/generate-*`, `scripts/vault-*`, Quartz `site/` | `web/*` | `docs/agents/web.md` |
| [`saas`](docs/agents/saas.md) | Cloud Run, Cloud SQL, GCE MQTT, Firebase Auth, future React app | `saas/*` | `docs/agents/saas.md` |
| [`coordinator`](docs/agents/coordinator.md) | Schemas, migrations, CI, infra, cross-cutting refactors, review + merge | `coordinator/*` or direct to main | `docs/agents/coordinator.md` |

**Find your scope doc and read it before touching files.** Scope docs name what's yours, what adjacent agents touch, and what to route through coordinator.

## Shared territory

No agent owns these. Changes here go through coordinator (Jason) — file a focused PR, don't edit autonomously:

- `verdify_schemas/` — cross-layer Pydantic contracts; touched by every agent
- `db/migrations/` — schema migrations; serialized, reviewed holistically
- `docker-compose.yml`, `systemd/`, `traefik/`, `mqtt/`, `.github/workflows/` — infra
- `CLAUDE.md` (this file), `README.md`, `docs/agents/**` — organizational docs
- `pyproject.toml` — tool config

Rule: if the file listed here is in your diff, pause and ask coordinator.

## How agents coordinate

1. **Schema changes land first.** If your work needs a new `verdify_schemas/` model or a field addition, land that in a schema-only PR (coordinator reviews). Next cycle, the consumer PR (yours) lands against the new schema.
2. **Migrations are serialized.** One migration PR at a time across the whole repo. Coordinator approves the sequence.
3. **When you need another agent's territory**, file a focused PR into their scope, don't reach across. Label it `requested-by: {your-agent}` in the PR body. The owning agent reviews on their next cycle.
4. **Drift guards are the wire protocol.** If `verdify_schemas/tests/test_drift_guards.py` passes, two agents can merge independently — the boundary is intact.
5. **Hand off by doc, not by DM.** Anything a future session of any agent needs to know goes into that agent's `docs/agents/{name}.md` or a memory file, not into chat.

## Branches & sprints

- Each agent has its own sprint counter. Example: `ingestor/sprint-5-...`, `firmware/sprint-7-...`, `saas/sprint-11-...`.
- The old dual-stream numbering retires. The prior operational sprints (17–23) are documented in each agent's scope doc where they overlap.
- Sprints land as **one commit per sprint** with a detailed multi-section message (see `e96f9ba`, `47f8154` for examples).

## Worktrees & memory

- Worktrees: `/mnt/iris/verdify-worktrees/{firmware,ingestor,genai,web,saas}/`. The `main` worktree at `/mnt/iris/verdify` is coordinator-only.
- Persistent agent memory: `~/.claude-agents/verdify-{agent}/projects/-mnt-iris-verdify-worktrees-{agent}/memory/`.
- User-level and feedback memories (about Jason, how he likes to work) are shared across all agent dirs — duplicate them at the start of each agent's life.

## Backlog

See `docs/BACKLOG.md` for the cycle index. Per-agent backlogs in `docs/backlog/{agent}.md`. Cross-cutting work (schemas, infra, Grafana, deps) in `docs/backlog/cross-cutting.md`.

## Checks before commit

- `make lint` (ruff) — required, no exceptions.
- `make test` — required; 1 pre-existing flaky timeout (`test_dew_point_risk_computes`) is tolerated, everything else must pass.
- `make firmware-check` — required for `firmware` agent only.
- For UI/site changes, verify render locally; type-checks and tests don't catch visual regressions.

## Firmware freeze rules (Phase 0 stabilization)

Post-2026-04-21 incident (sprint-15/15.1 fix-it-forward spiral producing repeated regressions). Background + full plan at `.claude-agents/iris-dev/plans/yo-iris-dev-you-help-humming-stonebraker.md`. These rules apply to every agent and every change to `firmware/lib/**`, `firmware/greenhouse/**`, `verdify_schemas/**`, `ingestor/entity_map.py`, or `mcp/server.py`.

1. **No firmware OTA deploy while any `severity ∈ {critical, high}` alert is open.** `make firmware-deploy` preflight queries the alerts table and aborts. Override requires operator sign-off in PR body.

2. **≤1 firmware OTA per calendar week** during rewrite phases (Phase 2-3 of the plan). Tunable pushes via `set_tunable` / `set_plan` are exempt but logged. Counter resets Monday 00:00 MDT.

3. **48-hour bake minimum** between firmware OTA deploys. `make firmware-deploy` preflight checks `firmware/artifacts/last-good.ota.bin` mtime. "Bake" = the new binary runs 48 hours without the sensor-health sweep flagging critical alerts.

4. **No sprint numbers.** Every change is PR-scoped and must carry replay-diff output + invariant-suite result + unit-test delta as artifacts in the PR description. Don't create `sprint-N.M` docs.

5. **Stress-window warning.** If outdoor_temp > 85°F forecast for the next 24 hours, `make firmware-deploy` reports it as operator context but does not block. Severe alerts, 48-hour bake, and weekly OTA limits remain hard gates.

6. **Every new tunable needs a `cfg_*` readback.** CI job `no-new-fire-and-forget` enforces this on PRs touching `firmware/greenhouse/tunables.yaml`. Fire-and-forget tunables are silent-push-corruption risks.

7. **Schema changes require explicit restart documentation.** If a PR touches `verdify_schemas/**`, `ingestor/entity_map.py`, or `mcp/server.py`, the PR body must mention which services need to bounce post-merge (`verdify-mcp`, `verdify-ingestor`). CI job `service-restart-drift-guard` enforces this. Observed need from the 2026-04-21 MCP staleness incident.

8. **Every firmware PR must show a replay-diff.** CI job `firmware-replay-diff` runs `scripts/firmware-replay-diff.sh` against merge-base. Default `THRESHOLD_PCT=0` means zero mode/relay divergence allowed. Intentional divergence (e.g. Phase 2 dwell-gate rollout) requires coordinator approval + explicit `THRESHOLD_PCT` override in the PR.

9. **Required PR artifacts** for firmware changes:
   - Replay diff output (`make firmware-replay OLD=<base> NEW=HEAD`)
   - Invariant-suite output (`make firmware-invariants`)
   - Unit-test delta (`make test-firmware`)
   - Coordinator (iris-dev) independent replay reproduction
   - Iris planner concurrence brief for any interface-level change

Coordinator approves merge only when all three reviewers (firmware agent, coordinator, iris) agree. Then 48-hour wait before OTA.

## Testing infrastructure (phase-0 deliverables)

- `make firmware-invariants` — runs the 15 bulletproof invariants (`firmware/test/invariants.h`) against the replay corpus. First breach fails.
- `make firmware-replay OLD=<ref> NEW=<ref>` — dual-worktree diff of firmware mode/relay decisions. Default THRESHOLD_PCT=0.
- `make replay-corpus-refresh` — pulls a fresh CSV from live DB, archives the prior corpus, validates no >5% size regression.
- `scripts/export-replay-overrides.sh` — CSV export includes outdoor sensors, equipment_state, mode_reason (sprint-15.1+).

---
> Source: [jrvallery/verdify](https://github.com/jrvallery/verdify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
