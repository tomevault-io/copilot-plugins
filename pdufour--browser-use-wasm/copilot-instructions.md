## accuracy-speed-tradeoff

> When accuracy improvements may trade against task-inference speed


# Accuracy vs speed trade-off bands

Default: accuracy changes must keep the green-circle metric (best `[perf:e2e] task inference` ms from `npm run test`) **flat (within ~5% noise)**.

Exception — **huge accuracy wins may be committed with a tiny slowdown**:

| Slowdown | Required accuracy gain | Verdict |
|---|---|---|
| ≤ ~5% (noise) | any gain beyond noise | ✅ commit |
| ~5–15% | **dramatic** — e.g. ≥ +5 pts strict bbox on Mind2Web @250, or comparable MiniWoB success jump | ✅ commit |
| > ~20% | n/a | ❌ never |

Requirements for every such commit:

- Measure BOTH sides honestly on the shared eval (Mind2Web @≥100 / MiniWoB filtered set) and `npm run test` (all cases green).
- Commit message must state both numbers: before→after accuracy AND task-inference ms.
- One change per commit so trades can be reverted individually.
- No query overfitting or fixture hacks to manufacture the "win" — see `no-grounding-query-overfit.mdc`.

Example message: `Mind2Web 39.6%→46.0% @250, task 2105→2310ms (+9.7%) — bicubic vision downscale`

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
