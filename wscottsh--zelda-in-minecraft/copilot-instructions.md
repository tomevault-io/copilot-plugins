## zelda-in-minecraft

> Issues live in this repo's GitHub Issues, driven by the `gh` CLI. See `docs/agents/issue-tracker.md`.

# zelda-in-minecraft

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues, driven by the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles, each label string equal to its name (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context — `CONTEXT.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Standing authorizations

These are durable grants from the repo owner. They override default agent caution — do not re-ask for them each session.

### Docs land on `main` directly

`CONTEXT.md` and `docs/adr/*` are prose, not code. Nothing breaks if they land; everything breaks if they sit unmerged, because the next session reads `main`, sees no ADR, and re-litigates a decision that was already settled.

So: commit them straight to `main` and push. No branch, no PR, no waiting to be asked. This is standing authorization — treat it as already granted.

If product code must accompany the decision, split it: docs to `main` now, code to its own branch and PR.

### Issue tracker writes are pre-approved

`gh issue` and `gh pr` writes — creating, editing bodies, labelling, commenting, closing — are normal operation for the wayfinder map. Don't ask.

### Still ask first

Force-pushing, history rewriting, hard resets, branch deletion, deleting issues, and anything touching a `research/*` branch's history.

---
> Source: [wScottSh/zelda-in-minecraft](https://github.com/wScottSh/zelda-in-minecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
