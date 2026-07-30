## mind2web-eval

> Mind2Web offline grounding eval — strict bbox, one query, no eval oracle


# Mind2Web grounding eval (offline benchmark)

**Not CI.** Gate regression stays `npm run test` (ShowUI-2B on ShopDemo). Mind2Web scores frozen dataset screenshots via the real app UI (`tests/mind2web/mind2web-grounding-eval.mjs`).

## Pass bar (strict — non-negotiable)

- **PASS** only when **bbox_acc** (click marker **inside** dataset `bounding_box_rect`) ≥ `MIND2WEB_EVAL_PASS_HIT_PCT` (default **85%**).
- **HIT** = marker inside scaled bbox (`edge=0`, `0.5px` tolerance on fractional scale).
- **NEAR** = parsed click within **25px** of bbox edge but **outside** the rect — **diagnostic only**; **never** counts toward pass.
- **MISS** = parsed but not HIT (includes NEAR).
- **FAIL** = capture/inference error or no parse. Early exit (opt-in `MIND2WEB_EVAL_FAIL_EARLY_PCT`) counts **harness** FAIL only — not MISS/NEAR or Find parse timeout on bad labels.

**Forbidden — do not re-add:**

- Multi-query Find, bbox-oracle “pick best label”, warm second Find, query sweeps
- `BIRTHDAY Day` / control-name synthesis, `isTechnicalLabel` heuristics, candidate ranking, or any tuning to lift Mind2Web %
- `near_acc` or edge-distance thresholds as pass criteria
- Weakening bbox (`BBOX_HIT_EPS_PX` stays `0.5`)
- `src/` hooks, DOM expected coords, or capture hacks for dataset rows

Report `near_acc` for analysis only.

## One query, one Find (production-faithful)

- **Goal:** Mind2Web `target_action_reprs` text (`[link] Reservations -> CLICK` → `Reservations`), else **first** visible attribute in fixed key order — no longest-label ranking.
- **CLICK** — fill Goal before Capture, then `btn-find` click only (gate `runE2EProductionJourney`); no quiet DOM injection, second fill, or `force` click.
- **TYPE / SELECT** — after Capture, structured voice tool call with `operation.value` (`type_into_field` / `select_option` via `__e2eVoiceTool`); `?e2e=1` only when those ops are in the run. Score field bbox from marker — DOM type/select on frozen shots may no-op.
- Uncheck `#browser-use-auto` so Find does not live-click the browse iframe.
- Skip fetch rows only when repr + attributes yield **no** label text (nothing to type).
- `MIND2WEB_EVAL_MAX_SRC_H` / `MIND2WEB_EVAL_MAX_BBOX_BOTTOM_FRAC` default **off** — opt-in scope only; do not tune defaults to lift %.

## Blackbox

| Allowed | Forbidden |
|---------|-----------|
| Playwright → Load, browse, Capture, Find | `import` from `src/` |
| `tests/e2e/e2e.js` helpers | Eval hooks, worker PNG injection |
| Node: HF fetch, bbox score | Live DOM rects for expected coords |

Coords from ShowUI on the **SnapDOM screenshot** only.

## Timeouts (eval-only)

Tall shots: nav/capture 90s/60s, prewarm 45s, per-Find 25s, page 180s. Do not raise gate E2E `INFERENCE_TIMEOUT_MS`.

## Related

- `docs/mind2web-eval.md`
- `blackbox-e2e.mdc` — CI gate (separate bar)
- `no-dom-grounding.mdc`

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
