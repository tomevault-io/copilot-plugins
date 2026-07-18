## scale-gds

> > **Category:** database · **License:** Public domain · **HIPAA:** none

# scale_gds — Claude Session Context

> **Category:** database · **License:** Public domain · **HIPAA:** none
>
> **Registry entry:** [meta/ecosystem.yaml#scale_gds](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/meta/ecosystem.yaml)

---

## Role in ecosystem

GDS-15 geriatric depression scale (15 items; PD).

## Current state

_Last refreshed: 2026-07-04 — run `creeden_enterprise scaffold scale_gds --refresh` to update._

<!-- preserve:start current_state -->
Registry marks this repo as **active**. Author a sentence or two describing the point-in-time state when this snapshot was refreshed — what works, what's partially done, what's pending. Use `creeden_enterprise scaffold scale_gds --refresh` to rerun the snapshot.
<!-- preserve:end current_state -->

## Active work

<!-- preserve:start active_work -->
No active work recorded — check this repo's open GitHub Issues and the Creeden Work Board.
<!-- preserve:end active_work -->

## Sync contract

None — this repo has no sync contract in `meta/ecosystem.yaml`.

## How to start a session

```bash
cd C:/Users/justi/code/active/scale_gds
git pull --rebase origin master
creeden_enterprise doctor scale_gds
```

## How to end a session

1. Commit and push on a `session/<description>` branch, then open a PR via `gh pr create --fill`.
2. From the `creeden_enterprise_hub` root: `creeden_enterprise sync --push` to propagate any ecosystem-level changes.
3. `creeden_enterprise drift` — reconcile if this session touched a plugin file (see **Sync contract** above).
4. Update this repo's `STATUS.md` — flip **Status**, bump **Last updated**, triage **Open issues**.

## Skills and rules

If this repo defines `.claude/skills/<name>/SKILL.md` files, the `name:` frontmatter values must not collide with any other active daughter's skills — the launcher's `--add-dir` set means peer-project skills load alongside this repo's, and `Skill()` resolves by first match in load order. See the canonical [skill-resolution rule](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/.claude/rules/skill-resolution.md) at the hub. `creeden_enterprise doctor` enforces uniqueness across the ecosystem.

**Operator hierarchy (per [ADR 0035](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/docs/adr/0035-multi-operator-topology-j-a-b-c-o.md)):** the ecosystem operates with five operator codes — `Operator J` (Justin, owner), `Operator A` (Claude Code, senior dev), `Operator B` (Codex, junior dev), `Operator C` (TBD competitor agent, reserved), `Operator O` (TBD gold-standard advisor, reserved). When authoring records, handoffs, ops-record PRs, or issues from this repo, use these codes. Cross-operator contract: [`docs/operations/operator-protocol.md`](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/docs/operations/operator-protocol.md) at the hub.

## Links

- Master repo: https://github.com/Creeden-Enterprise/creeden_enterprise_hub
- This repo in registry: [meta/ecosystem.yaml#scale_gds](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/meta/ecosystem.yaml)
- Handoff protocol: [docs/ecosystem/handoff-protocol.md](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/docs/ecosystem/handoff-protocol.md)
- Session-start protocol: [docs/operations/session-protocol.md](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/docs/operations/session-protocol.md)
- Skill resolution rule: [.claude/rules/skill-resolution.md](https://github.com/Creeden-Enterprise/creeden_enterprise_hub/blob/main/.claude/rules/skill-resolution.md)
- Gold-standard success criterion rule: shared `agents_conventions/AGENTS.md` (`## Working defaults` → `### Gold-Standard Success Criterion`); any local `.claude/rules/gold-standard-criterion.md` file is a pointer, not a duplicate source.

> Conventions: your user-global agent config (`AGENTS.md`). Todos/bugs → a labeled GitHub Issue. Cross-repo view → the Creeden Work Board.

---
> Source: [Creeden-Enterprise/scale_gds](https://github.com/Creeden-Enterprise/scale_gds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
