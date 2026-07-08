## aladiah-ai-spark

> **Required Reading:**

# CLAUDE.md — Aladiah repo

## Read before doing anything

**Required Reading:**

Read **`/docs/standards/LAUNCH_DECISION_PRINCIPLE.md`** at the start of every session.

This is Aladiah's root operating principle: **Hypothesis ≠ Fact. Evidence creates truth.**

Decision flow: Claim → Evidence → Classification → Priority → Work.

No blocker without proof. No hypothesis treated as fact. No work without ownership.

---

## Platform Canon

These five documents implement the root principle:

1. **NORTH_STAR.md** — why Aladiah exists (career transformation, not course completion)
2. **ARCHITECTURE_PRINCIPLE.md** — what qualifies to be built (serve ≥1 Core System, block 0)
3. **COMPETENCY_TAXONOMY.md** — approved competency slugs (single source of truth)
4. **QA_STANDARD.md** — DoR/DoD/Evidence gates (feeds blocker evidence into registry)
5. **LAUNCH_DECISION_PRINCIPLE.md** — root decision-making framework (required for all decisions)

If a request conflicts with the canon, surface the conflict before proceeding.

## Working rules (non-negotiable)

- No `.env` or live-DB writes without explicit approval.
- Production SQL is delivered as a reviewable file + paste-ready block; the human applies
  it by hand in Supabase. Claude Code does not auto-apply SQL.
- "Success / no rows" means the statement *ran*, not that it was *correct* — always follow
  a write with a verification `SELECT`.
- Verify structure before content; build before deploy.
- One discrete change at a time, verified. `/clear` between workstreams.
- Competency must be populated at insert time (never `null`) — competency is snapshotted
  onto attempt rows at submit and cannot be backfilled into past attempts.

## Repo facts

- Frontend: React/TypeScript/Vite on Vercel. Backend: Supabase (project
  `vgujnkxylipfwmkpwzvb`) + Railway (Node/Express).
- Build is `vite build` (esbuild, no `tsc`); `tsconfig.app.json` has `strict: false`.
- Quiz UI auto-prefixes `A)/B)/C)/D)` via `String.fromCharCode(65 + idx)` — never hardcode
  letter prefixes into option text.

---
> Source: [didierisonline-cell/aladiah-ai-spark](https://github.com/didierisonline-cell/aladiah-ai-spark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
