## open-record-replay

> Agent-facing operating guide for this repo. Humans: see [README.md](README.md), [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), [docs/TESTING.md](docs/TESTING.md), [CONTRIBUTING.md](CONTRIBUTING.md).

# AGENTS.md — Open Record & Replay

Agent-facing operating guide for this repo. Humans: see [README.md](README.md), [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), [docs/TESTING.md](docs/TESTING.md), [CONTRIBUTING.md](CONTRIBUTING.md).

## What this is

Open Record & Replay (`open-record-replay`) is a **privacy-first browser action recorder**. It records a controlled browser, turns actions into sanitized + classified steps, lets a human review/trim them, and exports a reviewable **skill draft bundle** that follows the [Agent Skills](https://agentskills.io) standard (loads on Claude Code, Codex, Kimi). **No video, no cookies, no screenshots, no raw secrets.** Generated bundles are DRAFTS requiring human review.

- Language: TypeScript, ES modules (`"type": "module"`), Node **20+**.
- Runtime deps: `zod` (schemas), `playwright` (capture). Tooling: `vitest`, `biome`, `tsc`.
- Local daemon serves the Recorder UI at `http://localhost:48321` (`RECORDER_PORT`).

## Commands (exact)

```bash
npm run dev          # run CLI from src via tsx (no build)
npm run build        # tsc + copy src/ui/index.html → dist (REQUIRED before npm start / demo bundles)
npm start            # run built CLI (dist/cli.js)
npm run lint         # biome check (src + tests)
npm run lint:fix     # biome autofix
npm run typecheck    # tsc --noEmit
npm test             # vitest: unit + tests/demos + tests/e2e
npm run test:e2e     # vitest: tests/e2e only
node dist/cli.js record --name "YOURSKILLNAME"   # build first; opens UI + starts capture
```

Verification gate before declaring work done / opening a PR:

```bash
npm run lint && npm run typecheck && npm test && npm run test:e2e && npm run build
```

## Architecture (map)

Full detail: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md). One-way dependency flow:

```text
cli → daemon → { capture, sanitizer, classifiers, review, storage, export }
                         └────────── all depend on → schemas (zod) ──────────┘
```

Pipeline: `RawEvent[]` (capture) → sanitize + classify (risk/value) → review/trim → approve → generate bundle.

| Area | File |
|---|---|
| Constants (`VERSION`, `RECORDER_PORT`) | `src/index.ts` |
| CLI | `src/cli.ts` |
| HTTP daemon + state machine (REST routes incl. skillify/open/zip) | `src/daemon/server.ts` |
| Browser capture (Playwright) | `src/capture/browser-capture.ts` |
| Sanitize raw→steps | `src/capture/sanitizer.ts` |
| Risk / value classifiers | `src/classifiers/risk-detector.ts`, `value-classifier.ts` |
| Zod data contracts | `src/schemas/{manifest,steps,review-state}.ts` |
| Local storage layout | `src/storage/local-store.ts` |
| Review/trim/approve | `src/review/review-manager.ts` |
| Bundle export + templates | `src/export/bundle-generator.ts`, `bundle-templates.ts`, `privacy-report.ts`, `replay-validator.ts` |
| Agent-executable specs + intent + pre-deploy | `src/export/agent-spec-generator.ts`, `intent-distiller.ts`, `pre-deploy-checker.ts` |
| Recorder UI | `src/ui/index.html` |

Recordings live at `~/.open-record-replay/recordings/<slug>/` (`raw.local.json`, `output/<name>/<name>/...` — the inner folder is the deployable, self-contained skill). Step types: `navigate | url_change | click | fill | select | key | wheel | pointer | submit_attempt | observe | select_result | handoff`.

## Where to change what

| Goal | Edit | Test |
|---|---|---|
| New low-risk value (region/terminal/date/count/query) | `classifiers/value-classifier.ts` | `src/classifiers/__tests__/value-classifier.test.ts` |
| New PII/sensitive pattern (always masked) | `classifiers/value-classifier.ts` | same |
| New risk boundary (payment/login/booking/CAPTCHA/delete) | `classifiers/risk-detector.ts` (+ `FORBIDDEN_ACTIONS` in `schemas/manifest.ts`) | `src/classifiers/__tests__/risk-detector.test.ts` |
| New/changed generated artifact | `export/bundle-templates.ts` → wire in `export/bundle-generator.ts` | `src/export/__tests__/` + `tests/e2e` |
| New daemon endpoint | `daemon/server.ts` (`requestPath === ... && method === ...`) | `src/daemon/__tests__/server.test.ts` |
| New demo | `tests/fixtures/<name>/` + `scripts/generate-demo-bundles.mjs` | `tests/demos/mvp-demos.test.ts` |

## HARD invariants — never break

These are the product's privacy/safety contract (enforced by Zod literals + pipeline logic). Do not weaken them to ship a feature.

- `capture.{screenshots,video,cookies,local_storage,session_storage,network_bodies} = false`.
- `privacy.{raw_values_sent_to_llm,screenshots_sent_to_llm,full_dom_sent_to_llm} = false`; `redacted_steps_only = true`.
- `sensitivity: "pii"` ⇒ `policy: "placeholder"` (never `stored`). `stored` needs `stored_value`; `placeholder` needs `placeholder`.
- `risk.final_submit_allowed = false`, `risk.payment_allowed = false`.
- High-risk action ⇒ emit a `handoff` step (`automation_allowed: false`); the sanitizer **stops** after it.
- Capture skips `type=password`, `type=hidden`, `autocomplete="cc-*"`.
- Bundle generation requires `review.approved === true`.
- Forbidden automation (`FORBIDDEN_ACTIONS`): `submit_payment`, `automate_login`, `bypass_captcha`, `use_unofficial_bypass`, `finalize_booking_without_user`.
- `tos.status` defaults to `unknown`; never claim ToS compliance unless a human verified it.

## Code conventions

- Biome: **no `any`** (`noExplicitAny: error`), avoid `.forEach` (`noForEach: warn`), double quotes, 2-space indent, 100-col, ES5 trailing commas, organized imports.
- TypeScript `strict` + `noUncheckedIndexedAccess` + `noImplicitReturns`; target ES2022, `NodeNext` modules.
- Vitest `globals: false` → import `{ describe, it, expect }` from `"vitest"`; reset stateful helpers (`resetStepCounter()`) in `beforeEach`.
- Never disable types with `as any` / `@ts-ignore`. Update the Zod schema first when data shape changes.

## Contribution rules

Small, reviewable PRs (classifier rule, value pattern, risk pattern, site preset, demo recording, reference template). Do **not** add video/screenshot/cookie/storage/network-body persistence, login/payment/CAPTCHA automation, ToS bypass, or automatic PR creation. For UI/daemon/export changes, attach evidence under `.sisyphus/evidence/`. Full rules: [CONTRIBUTING.md](CONTRIBUTING.md).

---
> Source: [Marker-Inc-Korea/open-record-replay](https://github.com/Marker-Inc-Korea/open-record-replay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
