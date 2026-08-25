## enteros-plugin-workflow-triage

> Provides the `/triage` slash command — a full on-demand PR-triage cycle

# molecule-ai-plugin-molecule-workflow-triage

Provides the `/triage` slash command — a full on-demand PR-triage cycle
comprising 7-gate verification, mechanical fixes, code review, cross-vendor
review, merge, docs sync, and issue pickup. Equivalent to one hourly cron tick,
triggered manually.

**Version:** 1.0.0
**Runtime:** `claude_code`
**Command:** `/triage`
**Skill:** `triage-workflow`

---

## What It Does

`/triage` runs the same workflow as the hourly cron tick, on demand:

1. **Activate guards** — load `careful-mode` REFUSE/WARN/ALLOW lists; replay last 20 cron-learnings.
2. **List open PRs and issues** — via `gh pr list` and `gh issue list`.
3. **7-gate verification per PR:**
   - Gate 1: CI status
   - Gate 2: Build
   - Gate 3: Tests
   - Gate 4: Security
   - Gate 5: Design review
   - Gate 6: Line review
   - Gate 7: Playwright (UI PRs only)
   - Supplement A: `code-review` skill
   - Supplement B: `cross-vendor-review` for noteworthy PRs (auth/billing/data-deletion/migration)
4. **Mechanical fixes on-branch** — fix Gate N failures with `fix(gate-N): ...` commits. Never fix logic/design/auth on-branch.
5. **Merge** — if all gates pass + 0 🔴 from code-review + cross-vendor agreement → merge (commit only, never squash).
6. **Docs sync** — invoke `update-docs` skill after any merge.
7. **Issue pickup (cap 2)** — gates I-1..I-6, branch + implement + draft PR + `llm-judge` ≥ 4.
8. **Status report + cron-learnings** — report + append 1–3 lines to cron-learnings JSONL.

### When to use

- Clear PR backlog faster than hourly cadence.
- Test changes to the triage prompt itself.
- Resume after a scheduled cron has died.

---

## Repository Layout

```
molecule-workflow-triage/
├── skills/
│   └── triage-workflow/
│       └── SKILL.md         # Full triage workflow definition
├── commands/
│   └── triage.md            # /triage command definition
├── adapters/
│   ├── __init__.py
│   └── claude_code.py       # Claude Code harness adapter
├── plugin.yaml               # Plugin manifest
└── README.md
```

---

## Key Conventions

| Topic | Convention |
|---|---|
| **Merge type** | Merge commit only — never squash or rebase |
| **On-branch fixes** | Mechanical fixes only (`fix(gate-N): ...`) |
| **Merge requirement** | All 7 gates + 0 🔴 + cross-vendor agreement |
| **Issue cap** | Max 2 issues picked up per triage run |
| **llm-judge threshold** | Score ≥ 4 to mark draft PR ready |
| **Gates that block** | Security (Gate 4), Design (Gate 5), code-review 🔴 |

---

## Standing Rules (Inviolable)

- Never push to `main`.
- `careful-mode` REFUSE list **always blocks**.
- Code-review 🔴 **always blocks merge**.
- Cross-vendor disagreement on noteworthy PRs **escalates to user**.
- `llm-judge` ≤ 2 **blocks marking a draft PR ready**.
- Never fix logic/design/auth issues on-branch — file a separate issue.

---

## Recommended Plugins

`/triage` calls these skills as sub-steps. Install them first:

```yaml
plugins:
  - molecule-skill-code-review      # Supplement A
  - molecule-skill-cron-learnings  # Step 5 learnings
  - molecule-skill-llm-judge       # Issue pickup scoring
  - molecule-skill-update-docs     # Docs sync after merge
```

---

## Development

```bash
# Dry-run: list open PRs
gh pr list --state open --json number,title,author,isDraft,mergeable,statusCheckRollup --limit 10

# Check current gate status
gh pr view 123 --json statusCheckRollup,mergeable,reviewDecision

# Verify careful-mode lists
cat ~/.claude/careful-mode-refuse.txt  # if using careful-bash
```

---
> Source: [EnterOS-AI/enteros-plugin-workflow-triage](https://github.com/EnterOS-AI/enteros-plugin-workflow-triage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
