## squads

> Orientation for a Claude Code session opened in this repo.

# CLAUDE.md

Orientation for a Claude Code session opened in this repo.

## What this repo is

`squads` is a documentation-and-content repo for **Agent Squads** — installable bundles
that deploy proactive sub-agents into a Pancake pod. It holds Pancake's official squads,
the source the Pancake marketplace seeds from, and the public contract for squad authors.

There is **no application code** here — no `package.json`, no build. Just zero-dependency
Node scripts: `scripts/validate.mjs` (mirrors marketplace ingestion),
`scripts/test-validator.mjs` (self-tests for the validator), and `scripts/eval.mjs`
(replay-tier workflow evals — see below).

## How the repo is organized

- `squads/<name>/` directories are **squad bundles** — the unit the marketplace ingests.
- `squads/<name>/evals/replay/<workflow-id>/*.trace.json` are **replay-tier evals** —
  recorded happy-path + negative-regression traces, replayed against the manifest by
  `scripts/eval.mjs`. Every bundle ships at least one happy-path trace per workflow;
  negative traces commit production incidents as living regression markers (the
  cofounder-hallucinated workflow id from the 2026-06-11 audit is one such trace in
  `analytics-squad/evals/`).
- `lib/eval-runner.mjs` is the shared check engine — no deps, JSON in / report out.
- `template/` is a complete, valid skeleton bundle authors copy from.
- `docs/` is the full contract and authoring guides.
- `scripts/validate.mjs` validates every bundle; `manifest.schema.json` is the editor schema.

## The bundle contract, in one paragraph

A bundle is a directory with a `manifest.json` (the package descriptor: `name`, `version`,
`description`, `author`, `agents` as a string array of ids, plus optional squad-wide
skills, required identities, and vault secrets), a `SQUAD.md` catalog card, an
`ONBOARD.md` onboarding script, and per agent: an `agents/<id>/agent.json` (the per-agent
runtime config — model, heartbeat, agent-specific skills, mirroring OpenClaw's
`agents.list[]`), `agents/<id>/IDENTITY.md`, and `agents/<id>/SOUL.md`. Optionally it
carries `MEMORY.md` seed memory, `skills/` files, `crons/jobs.json`, and a per-agent
`HEARTBEAT.md` (required when `agent.json` declares a heartbeat). Full detail is in
[`docs/bundle-reference.md`](./docs/bundle-reference.md).

## Working in this repo

- To **author** a new squad, use the `create-squad` skill — it includes a step for
  writing the bundle's first replay trace before declaring it finished.
- To **check or fix** a squad, use the `validate-squad` skill.
- To validate directly: `node scripts/validate.mjs` (add a bundle path to scope it).
- To run the replay evals: `node scripts/eval.mjs` (or scope to one bundle / one trace).
- Model new squads on [`template/`](./template/) (the skeleton) and the contract in
  [`docs/bundle-reference.md`](./docs/bundle-reference.md). Model new eval traces on
  any existing `*.trace.json` and the per-bundle `evals/README.md`.

Two CI gates, both blocking: `node scripts/validate.mjs` (bundle shape) and
`node scripts/eval.mjs` (workflow contract). A change that goes red on either is a
contract violation, not a flake — fix the bundle, not the gate.

### When you fix a production bug that surfaced in a workflow

Commit a **negative-case trace** for it alongside the fix. Set `expected: "FAIL"` plus
the `expected_failures: [...]` the contract check trips on. The runner inverts: exactly
those checks must fail, and if a future refactor silently un-catches the bug, the suite
goes red — the trace is the durable institutional memory of the incident. The two
`analytics-squad/evals/replay/posthog.daily_digest/regression-2026-06-11-*.trace.json`
entries are the worked example.

---
> Source: [get-pancake/squads](https://github.com/get-pancake/squads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
