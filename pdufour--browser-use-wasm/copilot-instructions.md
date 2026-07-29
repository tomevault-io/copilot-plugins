## browser-use-wasm

> This project follows strict engineering standards and specialized rules.

# Project Instructions

This project follows strict engineering standards and specialized rules.

## Rule Imports

The following rules are foundational mandates for this project:

- [No Source Document CSS Modification](.cursor/rules/no-src-css-mod.mdc)
- [Blackbox E2E](.cursor/rules/blackbox-e2e.mdc)
- [Client-Side Only](.cursor/rules/client-side-only.mdc)
- [Green Circle Commit](.cursor/rules/green-circle-commit.mdc)
- [LLM Intent Parsing](.cursor/rules/llm-intent-parsing.mdc)
- [No Demo Leaks](.cursor/rules/no-demo-leaks.mdc)
- [No DOM Grounding](.cursor/rules/no-dom-grounding.mdc)
- [VLA Registry](.cursor/rules/vla-registry.mdc)
- [Wllama Only](.cursor/rules/wllama-only.mdc)
- [Capture: No Demo Hacks](.cursor/rules/capture-no-demo-hacks.mdc)
- [Capture: No Visual-Diff Overfitting](.cursor/rules/capture-no-overfit.mdc)
- [Capture: Research-First Mandate](.cursor/rules/capture-research-first.md)

## Capture: Research-First Mandate

- **Experiments stay in Experiments** — Never use `src/capture-snapdom.js` as a scratchpad. All hypotheses MUST be tested using the experiment infrastructure (`scripts/snapdom-capture-experiments.mjs`).
- **Documentation First** — Findings, winners, and architectural decisions from experiments MUST be recorded in `docs/capture-alignment-experiments.md` or a dedicated ADR before implementation.
- **Generic Mechanisms Only** — Only ship changes to `src/capture-snapdom.js` that follow the "computed-style copy" or "uniform snap" patterns. No magic numbers or fixture-tuned offsets.
- **Verify against the bar** — A change is only ready for production when it hits the "Holy Grail" bar (medY <= 1 AND fullPageCorrelation >= 92%) across the target capture pool, or when the reason for the remaining drift is structurally documented.

## Capture: no visual-diff overfitting

Visual-diff / grid search on the built-in browse fixture is **hypothesis-only** — not license to overfit production capture to fixture scores. See [Capture: No Visual-Diff Overfitting](.cursor/rules/capture-no-overfit.mdc) and [Capture: No Demo Hacks](.cursor/rules/capture-no-demo-hacks.mdc).

- **No fixture overfitting** — grid winners on ShopDemo/header band do not ship unless the mechanism works for any site in `#capture-target`.
- **No magic numbers** — no global `line-height`, `yShift`, baseline overrides, pixel fudges, or tolerance tweaks chosen because they scored on one page.
- **Visual-diff wins ≠ global hacks** — a combo that tops the grid stays out of `capture-snapdom.js` unless it is generic, not fixture-tuned.
- **Ship generic mechanisms only** — computed-style copy live → clone per text node, uniform dimension snap from measured `#capture-target` rect, temporal freeze on clone, documented snapdom upstream fixes.
- **Visual-diff diagnostic** — legacy matrix runners removed; use `npm run visual-diff:bbox-visual` (`scripts/snapdom-bbox-visual-diff.mjs`). Never promote fixture-only hypotheses to production.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
