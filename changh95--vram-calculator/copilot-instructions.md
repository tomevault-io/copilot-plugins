## vram-calculator

> A single-page, fully client-side web app that answers one question for local-LLM

# VRAM Calculator

A single-page, fully client-side web app that answers one question for local-LLM
developers: **"Can I run model X on hardware Y?"** The user picks a model,
hardware (× quantity), quantization, context length, concurrent sequences, and
serving framework; the app computes required memory (weights + KV cache +
activations + overhead) and verdicts **Fits / Tight / Won't fit** against the
hardware's usable capacity.

## Stack & commands

- React + TypeScript + Tailwind + Vite. No backend — all math runs in the browser.
- `npm run dev` — local dev server (deployment is "local only" for now, but the
  build must always remain statically hostable).
- `npm run build` — static production build to `dist/`. Deploys to GitHub Pages
  (project site) via `.github/workflows/deploy.yml` on push to `main`; Vite
  `base` is `/vram-calculator/` for the build.
- `npm test` — Vitest. Run it before declaring any task done.

## Architecture rules

- **`src/lib/estimateVram.ts` is the heart of the product and must stay isolated.**
  Pure function(s) of `(config) -> MemoryBreakdown`; no DOM, no React imports.
  Each term (weights, KV cache, activations, runtime overhead, multi-device
  overhead) is a named, individually testable function.
- All model/hardware/framework facts live in typed data tables under
  `src/lib/data/` (`models.ts`, `hardware.ts`, `frameworks.ts`). UI components
  never embed numbers or math — they render what the lib returns.
- Every constant in the data tables traces to a source: a comment pointing at
  the relevant `notes/` doc (which carries the citation). **Never invent a
  number.** If a value is unverified, mark it `estimate: true` in the data and
  the UI must surface an "estimate" indicator wherever it influences a result.

## Math & data integrity (the product's credibility)

- **Per-model real architecture data, not parameter-count heuristics.** KV cache
  is computed from each model's actual config (num_layers, num_kv_heads,
  head_dim, attention type — GQA vs MLA, sliding-window where relevant) sourced
  from HuggingFace `config.json`. MoE models record total vs active params;
  weights use total params.
- **KV-cache precision is not a user control.** Each framework entry defines its
  default KV dtype; the assumption is disclosed in the UI tooltip and results.
- **One "concurrent sequences" control** (no separate batch size / concurrent
  users — continuous batching makes them the same thing for memory purposes).
- **Multi-device:** quantity selector ("× N"). Capacity aggregates; weights/KV
  shard with a small per-device replication overhead, labeled as an estimate.
  Multi-chip Tenstorrent SKUs (T3K, Galaxy) are single options with aggregate
  memory.
- **Unified memory** (Apple, DGX Spark, etc.): usable capacity is a per-device
  parameterized cap (e.g. macOS GPU-allocatable fraction), never raw total RAM;
  the cap is shown in the UI.
- Verdict thresholds: Won't fit (>100% of usable), Tight (>90%), Fits otherwise.
  Won't-fit shows the shortfall in GB.

## TDD — non-negotiable

Tests are written **before** implementation, everywhere:

- `estimateVram` and each term function: unit tests with known-good reference
  cases (published VRAM numbers for well-understood configs).
- Data tables: validation tests (every model has positive KV dims, MoE
  active < total, every hardware entry has capacity > 0, unified entries have a
  cap, every framework defines a KV dtype, etc.).
- UI components: rendering/interaction tests (verdict states, estimate markers,
  URL state round-trip).

## Scope (v1)

- **Models:** Qwen3, Gemma 4, GLM-5, DeepSeek, Kimi, Nemotron families —
  variants and configs are web-researched and verified, not recalled from
  training data. See `model_list.md`.
- **Hardware:** NVIDIA (consumer + datacenter + DGX Spark), Apple M1–M5
  (Base/Pro/Max/Ultra), AMD Instinct, Tenstorrent (N150, N300, T3K, WH Galaxy,
  P150, QB2, BH Galaxy). See `hardware_list.md`.
- **Frameworks:** vLLM, SGLang, llama.cpp, Ollama, TensorRT-LLM, MLX, and the
  Tenstorrent stack (vLLM on tt-metal / tt-inference-server). See
  `serving_framework_list.md`.
- **Positioning:** vendor-neutral community tool with first-class Tenstorrent
  coverage. No vendor favoritism in math or verdicts.
- **Out of scope v1:** training/fine-tuning memory, accounts/server-side state,
  explicit TP/PP/EP parallelism math (beyond the simple × N model), offloading
  (CPU/disk) math.

## UX requirements

- Live recalculation on every input change; sensible defaults so a verdict is
  visible immediately (never an empty state).
- Full config encoded in the URL for shareable links.
- Verdict never relies on color alone (icon + text); WCAG AA contrast;
  keyboard-navigable controls.
- UI is internationalized (English / Korean / Simplified Chinese / Japanese) via
  `src/lib/i18n.ts` — a top-right language dropdown, choice persisted to
  localStorage. Only UI chrome/labels/verdict are translated; the lib modules
  (`estimateVram`/`suggestions`) stay English and produce canonical English
  assumption/suggestion text. Add new UI strings as keys in `i18n.ts` (the
  `Record<TKey, string>` typing forces every language to define every key).
- Dark mode default with light toggle. Visual direction:
  `reference/index-mock.html` is the *feel* reference — dense, technical,
  developer-tool aesthetic — but the real UI should elevate it (use the
  frontend-design plugin), not copy it. The mock's formulas are placeholders;
  never port them.

## Docs map

- `notes/` — research with citations: `vram-formula.md` (the formula and each
  term's derivation), `hardware-constraints.md` (unified-memory caps, usable
  VRAM realities), `serving-framework-vram.md` (per-framework KV strategy and
  savings techniques + magnitudes).
- `model_list.md`, `hardware_list.md`, `serving_framework_list.md` — supported
  lists (repo root); regenerate/update these whenever the corresponding data
  tables change. Data tables and docs must never drift apart.
- `vram-calculator-design-prompt.md` — original UX spec;
  `reference/index-mock.html` — visual mock (reference only, superseded by the
  app).

---
> Source: [changh95/vram-calculator](https://github.com/changh95/vram-calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
