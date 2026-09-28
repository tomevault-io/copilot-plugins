## healthee

> An honest, self-hosted AI health companion built on data reverse-engineered from

# Healthee

An honest, self-hosted AI health companion built on data reverse-engineered from
an Amazfit Helio Strap. It never flatters: every interpretive claim cites a real
note in a graded research corpus and is calibrated to that note's grade, every
number carries its data confidence, and "not enough data" always beats an
optimistic guess. (What is *enforced* is citation, id-existence and
grade-calibration against the manifest — not entailment. There is no step that
reads a note and decides whether it supports the sentence, by choice and for
cost; `docs/INTELLIGENCE.md` section 3 says so plainly, and the three deterministic
surfaces that go further are named there.)

**This is the clean rebuild monorepo.** The previous implementation lives at
`~/projects/healthee-legacy` (read-only reference — mine it for the proven BLE
protocol layer, the validated science code, and hard-won gotchas; do NOT import
its structure or habits). Prod still runs from the legacy repo's remote until
cutover.

## Read before writing any code

- **docs/ENGINEERING_STANDARDS.md** — binding quality gates (400-line file max,
  no swallowed errors, tests in the same PR, science code is sacred). Diffs that
  violate a MUST get rejected in review even if they work.
- **docs/HOW_WE_VERIFY.md** — how we find out whether the code is what it
  claims: mutation testing and its four ways of lying, the honesty layer's
  structural enforcement, and the traps that have cost us real time.
- **docs/ARCHITECTURE.md** — target architecture + the phase plan.

## Working model

The lead agent (Fable) writes briefs and reviews all code; implementation agents
write the code. Nothing lands without review against the standards doc.

## Layout

```
apps/server/      Python 3.13 backend — FastAPI · psycopg3 · TimescaleDB
apps/mobile/      Flutter app — Riverpod · 60-day local store · on-device analytics
apps/landing/     public landing page — Astro · Tailwind v4 · static, self-contained
packages/knowledge/   graded research corpus (notes/ = the evidence base)
packages/contracts/   API contract snapshots + golden fixtures shared server↔mobile
infra/            docker · nginx · deploy · backup
docs/             architecture · standards
```

- **Server**: layout + rules in standards doc §2. All LLM calls go through the
  grounded-ask choke point (citation-validated against `packages/knowledge`).
- **Mobile**: layout + rules in standards doc §3. All dart-defines live in
  `core/env.dart`; single API client via provider; typed models at the data
  boundary (no raw maps in feature code).
- **Landing**: design system + binding content rules in `apps/landing/DESIGN.md`
  — the page is bound by the product's honesty contract (claim only what is
  built; competitor claims only from `docs/PRICING.md` §2; unbuilt features
  marked planned). Semantic color tokens only, no external fonts/CDNs/trackers.
- **Knowledge**: per-claim evidence grades with calibrated language
  (Established → state plainly · Probable → light hedge · Emerging → flag ·
  Contested → present as debated · Myth → correct gently); citations
  real-or-absent (verify primary sources BEFORE writing); mandatory Honesty
  section; directives blocks — safety-critical directives are hard guardrails
  in code, never overridable by the LLM.

## Commits & CI

Conventional Commits, small thematic batches, short-lived branches, hooks via
`scripts/setup-dev.sh`, CI gates on every push — all in **CONTRIBUTING.md**.

## Hard rules (learned the expensive way in the legacy repo)

- **Never** add a `Co-Authored-By: Claude` trailer to commits.
- Commit in small thematic batches; one reviewable concern per commit.
- ONE canonical definition per metric — this is health data; two definitions of
  "sleep debt" is a lie waiting to surface.
- No composite scores without documented methodology and a research note.
- Calories: MET-by-state model, never Keytel/HR-EE for free-living.
- Science functions port verbatim from legacy; behavior changes are their own
  PR with known-value tests.
- `git push` BEFORE any prod deploy (VPS deploys via `git reset --hard origin/…`).
- psycopg3 `execute()` won't run multi-statement SQL — one statement per
  execute; migrations are committed modules, not pasted shell.
- Scrollable chart screens use `ListView.builder` + reveal-once animation, or
  charts replay on every scroll.
- App build config via `--dart-define`: `HELIO_API`, plus `SUPABASE_URL` and
  `SUPABASE_ANON_KEY` for a build that can sign in. The **anon** key, never
  `service_role` — that one bypasses every policy and belongs on the server.
  A build without the two Supabase defines still works: it offers the
  transitional pasted-token form and says why. `AUTHKEY`/`MAC` are NOT defines —
  they are per-owner secrets in the keystore (`core/env.dart` says why), and
  neither is a bearer token: since the Supabase sign-in landed, the app mints
  its own device token per phone.

---
> Source: [afkcodes/healthee](https://github.com/afkcodes/healthee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
