## damdam

> Read [`/docs/README.md`](./docs/README.md) first for the full spec

# DamDam — Agent Instructions

Read [`/docs/README.md`](./docs/README.md) first for the full spec
index and recommended reading order. All product, technical,
security, and infrastructure decisions live in `/docs` — treat it
as the source of truth, not this file.

## Before implementing any feature

1. Find the User Story (`US-XX`) in [`/docs/prd.md`](./docs/prd.md)
   §4 and its acceptance criteria
2. Check [`/docs/testing-qa.md`](./docs/testing-qa.md) for the
   required test cases mapped to that story — write these before
   or alongside the implementation for the strict-TDD categories in
   §14.1 (check-in, SOS, payment/idempotency, auth)
3. Cross-check the relevant sections in `/docs/data-model.md`,
   `/docs/api-spec.md`, and the frontend spec for your platform
   (`/docs/frontend-mobile.md` or `/docs/frontend-dashboard.md`)

## Who implements what

This project splits work by **concern, not by whole codebase area**
— the split exists to put the small amount of genuinely taste-
sensitive work where design judgment matters, and everything else
where it's cheapest to build, without defaulting to "Claude builds
all of mobile" (that would be far more Claude usage than necessary
and works against the cost-optimization policy below).

**The core mechanism: Claude produces a design system once, Codex
implements against it, Claude reviews the rendered result.**
Codex's frontend gap is a taste/judgment gap, not a capability gap
— it's very capable of assembling already-decided design tokens,
component patterns, and layout rules precisely. Removing ambiguous
taste calls from Codex's job (by giving it an explicit system to
follow) closes most of the quality gap without Claude touching
every screen.

| Area | Primary implementer | Why |
|---|---|---|
| `docs/design-system.md` (tokens, typography, spacing, component patterns) | **Claude**, produced once, revised rarely | The one-time investment that makes everything below cheaper to get right |
| `apps/mobile/src/screens/SosConfirm/**`, `SosSent/**`, and the onboarding flow (`Splash` through `DepartureDate`) | **Claude, direct implementation** | The small, named exception — these are the screens where the *feel* of the interaction (reassurance during a real emergency, trust during first use by a low-tech-confidence demographic) matters more than typical UI, and a taste gap here has outsized real-world cost |
| All other `apps/mobile/src/screens/**`, `apps/mobile/src/components/**` | **Codex**, against `design-system.md` | Implemented by assembling the design system's already-decided pieces — Claude reviews the rendered output (see the GitHub Action below), doesn't build it |
| `apps/mobile/src/hooks/**`, `apps/mobile/src/services/**`, `apps/mobile/src/store/**` | **Codex** | Logic/state layer — no visual output |
| `apps/dashboard/**` | **Codex** | Explicitly speced as "functional, not polished" in `frontend-dashboard.md` §9.5 — the taste gap doesn't matter here |
| `apps/api/**`, `scripts/`, Terraform/infra | **Codex** | Backend/infra — no UI surface |
| QA/review across all of the above | **Claude** | See the GitHub Action below |

**If you are Codex implementing a mobile screen:** read
`design-system.md` first and build strictly against it — don't
improvise spacing, color, or component choices not covered there;
flag a gap in the PR description instead of guessing, so Claude's
review catches it as a design-system omission to fix once, not a
one-off judgment call to repeat.

**If you are Claude reviewing a mobile screen Codex built:** don't
just review the code — actually render the screen (or request a
screenshot in CI, see the GitHub Action) and check it against
`design-system.md` and the per-screen spec in `frontend-mobile.md`.
A screen that passes its tests but looks generic or inconsistent
with the system is a real review finding, not a nitpick.

**If you are Codex working anywhere in `apps/mobile`'s logic
layer:** stay inside hooks/services/store — don't restructure
screens or components even if it would simplify your logic; flag
it as a suggestion instead and let the screen's assigned
implementer (Codex-against-system, or Claude for the named
exceptions) make that call.

## Build & test commands

```
# API (FastAPI)
cd apps/api && pytest --cov=app
cd apps/api && ruff check . && mypy app

# Mobile (React Native, iOS + Android)
cd apps/mobile && npm test
cd apps/mobile && npm run lint && npm run type-check

# Dashboard (Next.js)
cd apps/dashboard && npm test
cd apps/dashboard && npm run lint && npm run type-check

# Full local stack
docker compose up
```

## Non-negotiable conventions

- Every feature PR references its User Story ID (`US-XX`) from
  `prd.md` in the PR description
- Data model changes require a numbered amendment in
  `data-model.md` (see §6.4/§6.5 for the pattern), not a silent edit
- API changes must keep `api-spec.md` in sync — CI fails on drift
  between the committed spec and FastAPI's generated OpenAPI output
- Check-in and SOS features (`US-15`, `US-16`) are safety-critical:
  any change touching the offline queue, sync retry logic, or
  notification dispatch requires the full test suite in
  `testing-qa.md` §14.4 (offline/chaos scenarios), not just unit
  tests, before merge
- Never weaken the offline-first write-before-network pattern
  (`prd.md` §5.6) to "simplify" a feature
- **All third-party vendor integrations go through an internal
  abstraction layer — `OTPService` (Termii primary/Twilio Verify
  secondary underneath, per `prd.md` §5.1), `VoiceService`/
  `VoiceProvider` (Telnyx underneath, with an IDT Express BYOC
  migration planned post-MVP per `prd.md` §5.5), `ESIMService`
  (Monty Mobile/eSIM Access/1Global underneath, three-way cascading
  failover), and `PaymentService` (Paystack/Flutterwave underneath)
  — never called directly from route handlers.** The API layer is
  already vendor-agnostic (`api-spec.md` §7.1/§7.4/§7.5 don't leak
  vendor-specific shapes into the contract), and `data-model.md`'s
  `aggregator` and `processor` enums already anticipate multiple
  providers per package/transaction (§6.6, §6.7). This convention
  makes switching or adding a vendor (Telnyx-to-IDT BYOC, a new
  eSIM aggregator, a payment processor fallback) a configuration
  change inside one service, not a rewrite across the codebase —
  this matters concretely here since all four vendor decisions
  (OTP, voice, eSIM, payments) are now resolved but held as
  configuration rather than hardcoded, precisely so a future change
  doesn't require this kind of rewrite.

## Cost-optimization policy (Claude usage)

Claude Code runs on direct Anthropic API billing for this project
(not the Claude.ai subscription), so every invocation has a real
marginal cost. Codex runs under a ChatGPT Plus subscription with no
comparable per-task cost. Optimize accordingly:

- **Default to Sonnet, not Opus**, for routine review and mobile UI
  implementation. Only escalate to Opus for a genuinely hard
  architecture call, and do it explicitly (e.g., comment "@claude
  use opus" on a PR), not by default.
- **Codex handles everything it's assigned above without
  escalating to Claude** unless the task is explicitly one of the
  strict-TDD categories in `testing-qa.md` §14.1, or touches
  `apps/mobile/src/screens`/`components`.
- **Don't paste large spec sections into a Claude prompt manually**
  — let Claude's file-read tools pull `/docs` content directly, so
  repeated reads of stable files benefit from prompt caching rather
  than re-billing full-price tokens every session.
- **Keep Claude sessions scoped to one PR or one feature at a
  time** rather than one sprawling session touching many unrelated
  files — this both controls cost and keeps review quality focused.
- **Output-token compression (`JuliusBrussee/caveman`) is scoped to
  `claude-review.yml` only, not interactive sessions.** That
  workflow's output is a structured checklist where terse is
  correct UX; ~65% output token reduction there is a real, low-risk
  win. Do not apply it to local interactive `claude` sessions —
  detailed reasoning in architecture/QA discussion has repeatedly
  caught real spec inconsistencies in this project, and compressing
  it would cost more in missed issues than it saves in tokens.

## GitHub Action — Claude Review

`.github/workflows/claude-review.yml` runs Claude automatically on
every opened/ready PR and on `@claude` mentions in PR comments. It
is scoped to Sonnet with a turn limit for cost predictability, and
its prompt already routes review emphasis correctly (visual/UX
review for `apps/mobile` screens/components, TDD/offline-scenario
verification for safety-critical logic, spec-drift/cross-tenant
checks otherwise). Codex's own "Automatic reviews" setting can stay
on in parallel as a fast first-pass P0/P1 filter — it should not be
treated as a substitute for Claude's review on anything the table
above assigns to Claude for review.

## Commit convention

Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`) — see
root `README.md` for branch strategy.

## What NOT to do

- Don't invent new API endpoints without adding them to
  `api-spec.md` first
- Don't add dependencies without checking `aarch64` wheel/binary
  availability if the change touches anything deployed to the OCI
  Ampere A1 instance (see `scaling-infrastructure.md` §12.2)
- Don't touch `.env.production` or any file under
  `/apps/api/secrets/` — these are runtime-only, never committed
- Don't mark a PR ready for review without running the linked
  test cases from `testing-qa.md`, not just "tests I wrote for
  this change"
- Codex: don't implement a mobile screen without reading
  `design-system.md` first, and don't touch the named exception
  screens (SOS confirm/sent, onboarding flow) — see the ownership
  table above
- Don't start building mobile screens at all until
  `design-system.md` exists — it's a prerequisite, not an
  optional nice-to-have (see "Who implements what" above)

---
> Source: [block-sig-hash/damdam](https://github.com/block-sig-hash/damdam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
