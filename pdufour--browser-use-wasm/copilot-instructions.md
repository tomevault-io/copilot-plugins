## no-grounding-query-overfit

> No query rewrites, candidate sweeps, or coord scoring for navigation


# No navigation query overfitting

**Client-side inference only** — see `no-dom-grounding.mdc`. Coords come from **one** ShowUI navigation inference on the **SnapDOM screenshot** (model-card `_NAV_SYSTEM` prompt), not from post-hoc query tricks or layout heuristics.

## One task, one inference

| Path | Task string |
|------|-------------|
| **Run task UI** | User Goal text verbatim (trim only) |
| **Voice tools** | Tool `target` / phrase from the voice controller — **not** rewritten option text |

Run inference once per user action. No fanout.

## Forbidden in `src/` (production and eval-facing paths)

- **Query rewriting** — mapping weak labels to other strings (`11` → `select 11`, option → `select ${option} in ${target}`, `click 12` → `calendar 12`, etc.)
- **Label candidates / query sweeps** — trying multiple task strings and picking a “winner”
- **Layout scoring** — aspect-based point ranking, “left column” / “header band” heuristics to choose among multiple model outputs
- **Mind2Web- or fixture-specific label hacks** — special cases for MTA, Budget, calendar cells, `LO`, junk field names `0` / `1` / `00`
- **Eval-oracle logic in product code** — bbox distance, NEAR bands, or dataset attributes to select queries or coords (harness stays in `tests/mind2web/mind2web-grounding-eval.mjs` scoring only — see `mind2web-eval.mdc`)

## Allowed (generic product fixes)

- **Model load / vision** — `n_ctx`, vision tokens, JPEG quality, tall-page **generic** resize in `snapdom/vision-resize.ts` (not per-site tuning)
- **Shared system prompt** — the card-verbatim `_NAV_SYSTEM` template in `actions/navigation.ts`
- **Capture pipeline** — generic SnapDOM fixes in `src/snapdom/capture.ts` (no fixture-tuned hacks — `capture-no-demo-hacks.mdc`)

## If accuracy is poor

Fix worker/WASM, vision resize, capture quality, or prompts — **not** more label variants or coord pickers.

When tempted to add `resolveGroundingLabel`-style helpers: **stop**. That path overfits Mind2Web noise and breaks production voice/E2E semantics.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
