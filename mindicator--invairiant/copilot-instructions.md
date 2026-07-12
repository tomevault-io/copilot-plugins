## invairiant

> invAIriant is an evidence-based, multi-lens architecture-audit protocol for

# AGENTS.md — invAIriant

invAIriant is an evidence-based, multi-lens architecture-audit protocol for
AI-era codebases. It is the **same protocol for any coding agent** — Claude
Code, Codex, Cursor, or otherwise. This file is the portable entrypoint that
makes it work without Claude Code's skill mechanism. The canonical, full
definition is [`skill/SKILL.md`](skill/SKILL.md); read it for the complete
runbook.

## The rules (non-negotiable)

```text
No evidence, no finding.
Do not average away critical risks.
Separate observation from verified finding.
Do not produce confident claims from vibes.
```

## When to run an invAIriant audit

When the user asks for an architecture / invariant / lens audit, an
evidence-based review, a PR audit, a phase-transition or post-incident audit —
or types one of these commands — run the pipeline below:

- `audit-pr [<range>] [--lenses a,b]` — a PR/diff: checklist + ≤2 focused lenses
- `audit-range <A..B>` · `audit-commit <sha>` · `audit-module <path>` ·
  `audit-adr <adr.md>` · `audit-rp <rp.md>` (refactoring proposal ↔ code) —
  same pipeline, different **pinned scope** (see below)
- `full-audit [<range>]` — whole repo, all mandatory lenses
- `verify-findings` · `classify-severity` · `synthesize-report` — single stages
- `closure-verification` — re-verify claimed fixes closed (no re-search)

Every audit resolves to one **audit target** = *pinned scope + evidence bundle
+ selected lenses + report type*; the scope commands differ only in the first
term. `invairiant collect --scope <pr|working|range|commit|module|adr|rp|repo>
[target]` resolves the pin to a **bounded** file set and **fails closed** if it
cannot — it never silently widens to the whole repo (that is `full-audit`,
chosen deliberately). `--scope pr --pr <N>` is the one **optional adapter** (may
reach the remote via `gh`/pull-ref; PR → `base...head` range); all other scopes
are pure-local. invAIriant audits **bounded engineering scopes, not vibes**;
audit only inside the bundle's `resolved_scope`.

## How to run it (the four-stage pipeline)

```text
[1] lens pass          prompts/lens-auditor.md        (one lens per pass)
[2] evidence verify    prompts/evidence-verifier.md   (adversarial: refute)
[3] severity classify  prompts/severity-classifier.md (rules, not averages)
[4] synthesize         prompts/report-synthesizer.md  (rejected items kept)
```

1. Read `invairiant.config.yml` (mandatory lenses, canonical docs, risk
   assets). Pick lenses by risk surface — **4–6, not 20**.
2. Build the evidence bundle: `invairiant collect --out .invairiant/cache/bundle.json`.
   Everything in it is a **candidate pointer, never a finding**.
3. Run one **lens pass per lens** using that lens file's `## Prompt Block`
   (under `lenses/`). Then verify → classify → synthesize.
4. Emit the report (`templates/audit-report.md`), render a PR comment with
   `invairiant render-comment`, gate with `invairiant ci-gate`, and
   `invairiant record` it to audit memory.

Stage boundaries are load-bearing: stage 1 never assigns final severity; stage
2 never invents findings; stage 3 touches only verified findings; stage 4 never
drops a rejected hypothesis. **Humans own the gates** — never merge/approve on
the basis of an audit.

The `invairiant` CLI (`cli/invairiant.py`, or `pip install -e .`) is a
judgment-free **seatbelt**: `init`, `collect`, `validate-config`,
`validate-report`, `render-report`, `render-comment`, `ci-gate`, `record`,
`history`. It runs no lenses and produces no findings.

Full runbook: [`skill/SKILL.md`](skill/SKILL.md) · lenses:
[`docs/lens-taxonomy.md`](docs/lens-taxonomy.md) · CLI:
[`docs/cli.md`](docs/cli.md) · worked demo: [`docs/demo.md`](docs/demo.md).

## Enabling invAIriant in your own repo

- **Claude Code:** symlink the skill — `ln -s "$PWD/skill" .claude/skills/invairiant`.
- **Codex (and any AGENTS.md-aware agent):** copy this file's rules +
  "How to run it" into your repo's `AGENTS.md`, and install the CLI.
- **Cursor:** this repo ships [`.cursor/rules/invairiant.mdc`](.cursor/rules/invairiant.mdc);
  copy it into your repo's `.cursor/rules/`.

See [`skill/README.md`](skill/README.md) for per-agent install details.

## Contributing to this repository

- **Commits are authored by the maintainer alone.** Do **not** add
  `Co-Authored-By` / `Assisted-By` trailers naming an AI agent — the local
  `commit-msg` hook rejects them.
- Before pushing, `python3 scripts/validate_framework.py` and the CLI smokes in
  `.github/workflows/validate.yml` must pass.
- Keep the CLI judgment-free; keep the skill the single source of truth for the
  audit procedure.

---
> Source: [mindicator/invAIriant](https://github.com/mindicator/invAIriant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
