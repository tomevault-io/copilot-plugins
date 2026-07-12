## invairiant

> invAIriant — evidence-based multi-lens architecture audit. Apply when the user asks for an architecture / invariant / lens audit, a PR audit, an evidence-based review, an audit of a commit range / single commit / module / ADR↔code drift / refactoring proposal, or types audit-pr / audit-range / audit-commit / audit-module / audit-adr / audit-rp / full-audit / verify-findings / classify-severity / synthesize-report / closure-verification.


# invAIriant audit protocol (Cursor rule)

Run architecture audits under one rule: **no evidence, no finding.** AI may
propose hypotheses; only evidence-backed, adversarially-verified claims become
findings. A high average score never cancels a critical finding. Keep
observations/hypotheses separate from verified findings.

**Procedure** — read [`AGENTS.md`](../../AGENTS.md) and, for the full runbook,
[`skill/SKILL.md`](../../skill/SKILL.md). In short:

1. Read `invairiant.config.yml`; pick 4–6 lenses by risk surface (not 20).
2. `invairiant collect --scope <pr|working|range|commit|module|adr|rp|repo>` → a
   **bounded** evidence bundle (candidate pointers, not findings); it fails
   closed if the scope can't be bounded, and audit only inside `resolved_scope`.
   `--scope pr --pr <N>` is the one optional adapter (gh/pull-ref → base...head
   range); the rest are pure-local. Bounded engineering scopes, not vibes.
3. Per lens, run its `## Prompt Block` (under `lenses/`): lens pass → verify →
   classify severity → synthesize a report (`templates/audit-report.md`).
4. `invairiant render-comment` for a PR comment; `invairiant ci-gate` to gate;
   `invairiant record` to audit memory. **Humans own the gates.**

The `invairiant` CLI is a judgment-free seatbelt (validate / collect / render /
gate / record) — it never runs a lens or produces a finding.

---
> Source: [mindicator/invAIriant](https://github.com/mindicator/invAIriant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
