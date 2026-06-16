## weightroom

> This file describes the codebase conventions and invariants that AI agents must follow when contributing to this project.

# Agent Guidelines

This file describes the codebase conventions and invariants that AI agents must follow when contributing to this project.

## Commands

```bash
npm run dev                # start dev server (http://localhost:5173)
npm run build              # TypeScript check + Vite production build
npm run typecheck          # tsc -b (no emit) — fast type-only check
npm run lint               # ESLint
npm run lint:fix           # ESLint with --fix
npm run test:unit          # Vitest — fast, no browser required
npm run test:unit:coverage # Vitest with v8 coverage (./coverage)
npm run test:e2e           # Playwright — requires dev server or build
npm test                   # both test suites
```

Always run `npm run test:unit` after changing anything in `src/lib/` or `src/hooks/`. Run `npm run test:e2e` after changing UI components. Run `npm run lint` AND `npm run typecheck` before finishing.

A pre-commit hook (`.husky/pre-commit`) runs `lint-staged` (eslint --fix on staged files), `tsc -b`, and the unit suite. Do not bypass with `--no-verify`.

## Architecture

The project is a pure client-side React app. No backend, no API routes.

**Core data flow:**
1. User selects model/quant/context in `ConfigCard`
2. `useCalcResult` (`src/hooks/`) calls `calcLLMRam()` → RAM breakdown
3. `calcDisk()` → storage breakdown
4. `useValueScore` (`src/hooks/`) calls `calcValueScore()` → TPS and cost-efficiency (requires hosting data)
5. Results render in `ResultCard` and `AvailableHardware`
6. State serialized to URL via `encodeState`/`decodeState` (500ms debounce)

## Key Invariants — Do Not Break

### KV cache formulas (`src/lib/calculator.ts`)

There are 4 KV formulas. Each must stay consistent across `calcLLMRam` and `calcValueScore` — both functions implement the same switch. If you change a formula in one, change it in the other.

| Formula | Used by |
|---|---|
| `standard` | Llama, Qwen, Mistral, Phi |
| `hybrid` | Gemma 2/3, Mistral Sliding Window |
| `mla` | DeepSeek V3/R1 |
| `linear_hybrid` | Qwen 3.5 |

### q1 weight overhead factor

```ts
const weightOverhead = quant === "q1" ? 1.0 : 1.1;
```

For Q1 (1-bit / MLX format) the overhead is already baked into `QUANT_BITS["q1"] = 1.25`. All other quants use 1.1 to account for embeddings and norms stored in higher precision. Do not remove this distinction.

### QUANT_BITS vs QUANT_BYTES

`QUANT_BITS` (in `quants.ts`) is used for RAM/disk calculations and works in integer-like bits.
`QUANT_BYTES` (in `calculator.ts`) is used for TPS/bandwidth calculations and uses fractional bytes per parameter.
They are separate for a reason — do not merge them. They MUST stay in sync though:
`QUANT_BYTES[q]` ≈ `QUANT_BITS[q] / 8` for every `q`. There is a parameterised
test in `calculator.test.ts` ("QUANT_BYTES matches QUANT_BITS / 8") that
will fail loudly if drift sneaks in.

### Quantization families & engine compatibility

`QUANT_SPECS` in `quants.ts` is the single source of truth for every quant we
support. Each entry carries a `family` field (`float | gguf | gptq | awq | mlx`)
which drives two things:

1. UI grouping in the Weights Quant dropdown (`getWeightQuantGroups`).
2. Inference-engine filtering (`QUANT_FAMILY_ENGINES`) — picking GPTQ/AWQ
   hides `llamacpp`; picking GGUF/MLX hides `vllm` / `tensorrt`. The full
   matrix lives in `QUANT_FAMILY_ENGINES` and `"custom"` is universally
   compatible (escape hatch for niche runtimes).

When adding a new quant: update `QUANT_SPECS` AND `QUANT_BYTES` (calculator.ts)
in one PR, otherwise the bits/bytes invariant test will fail. If the quant
introduces a new family, also extend `QUANT_FAMILY_ENGINES` and the family
labels documented in `QuantSelector.tsx` / `Footer.tsx`.

Effective bpw rules of thumb (see comments on `QuantSpec.bpw`):
- GPTQ g128 asym: bits + 0.25 (FP16 scale + zero point amortised over 128)
- AWQ g128:       bits + 0.25 (FP16 scale + scaled zero)
- MLX g64:        bits + 0.5  (FP16 scale + bias, smaller groups → more overhead)
- GGUF Q*_K_M:    bits exactly (block scales already counted in upstream specs)
- Q1 (sign-bit):  1.25 bpw (scale baked into QUANT_BITS — see q1 overhead note)

### URL state encoding

`encodeState`/`decodeState` in `src/lib/state.ts` use UTF-8 → base64url (no `+`, `/`, `=`). The encoding must remain stable — any change that breaks decoding of existing URLs is a breaking change for shared links.

### parseHfUrl

The regex must stop at `?`, `#`, and whitespace. Do not simplify it to `[^/]+` — that would include query params in the repo ID and cause 404s on HF API calls.

### `engineId` ⇄ `kvCacheFillPct` synchronisation

`ModelSettings.engineId` (a stable string id like `"llamacpp"` / `"vllm"` / `"tensorrt"` / `"custom"`) and `ModelSettings.kvCacheFillPct` (the numeric value used by `calcLLMRam` / `calcValueScore`) must stay in sync. The parent (`ConfigCard`) is responsible:

- selecting an engine preset → update **both** `engineId` AND `kvCacheFillPct` in one `setState`
- typing in the manual % input → update `kvCacheFillPct` AND stamp `engineId: "custom"`
- changing `quant` to a different family → call `pickCompatibleEngine` and, when it returns non-null, atomically update `quant` + `engineId` + `kvCacheFillPct` in a single `updateModel` call (auto-snap)

`resolveActiveEngine` in `src/lib/enginePresets.ts` is the single source of truth for "which preset is active" — if `engineId` matches a preset but `pct` disagrees, it falls back to Custom rather than silently lying with the wrong label. Keep that behaviour.

`pickCompatibleEngine` lives in the same module and decides whether the
current engine survives a quant change. Its contract: returns `null` when
no snap is needed (current engine is already compatible OR `engineId` is
`"custom"`), otherwise returns the first compatible preset's id+pct.
Tested in isolation (`ConcurrentUsersInput.test.tsx`).

`engineId` is intentionally optional in `ModelSettings` for backward compatibility with shared URLs created before this field existed; the UI falls back to pct-based matching when it is `undefined`. Do not make it required.

### `hardwarePresetId` ⇄ `HostingData` fields synchronisation

Hardware presets (`src/lib/hardwarePresets.ts`) mirror the engine-preset architecture but live in `HostingData`, not `ModelSettings`:

- `PRESET_OWNED_FIELDS` is the allow-list of `HostingData` keys a preset may write: `cpuModel`, `ramType`, `ramBandwidthGBs`, `availableRam`, `gpuCount`, `gpuVram`, `gpuBandwidth`, `gpuInfo`, `efficiency`. Every preset MUST write all of these explicitly (zero/empty for the branch it doesn't apply to) — the unit test enforces full coverage. Half-written presets leave stale Apple `cpuModel` next to a freshly-applied H100 in practice.
- selecting a hardware preset → spread `preset.hosting` over current `HostingData` AND set `hardwarePresetId: preset.id` in one `setState` (`AvailableHardware.handlePresetSelect`). Apple presets zero the discrete-GPU block (`gpuCount/gpuVram/gpuBandwidth=0`, `gpuInfo=""`) and snap `efficiency="60"` (Unified). GPU presets clear the Apple unified-memory block (`cpuModel/ramType/ramBandwidthGBs/availableRam=""`) and snap `efficiency="80"` (GPU).
- manually editing any field in `PRESET_OWNED_FIELDS` → stamp `hardwarePresetId: "custom"` so the dropdown switches to Custom (`AvailableHardware.update` does this automatically — keep that branch). This includes the BW efficiency preset buttons: clicking CPU/Unified/GPU after picking a card releases the preset.
- editing fields outside that allow-list (`osOverheadGb`, `price`, `notes`, `availableStorage`, `cpuCores`, `cpuFreqGHz`, `storageType`) → DO NOT touch `hardwarePresetId`; those fields are truly orthogonal and survive every preset switch.

`resolveActiveHardware` is the single source of truth for "which hardware preset is active" — same fail-loud semantics as `resolveActiveEngine`. Hardware and engine presets are orthogonal: picking an H100 must never touch `engineId`, and switching to vLLM must never touch `HostingData`. Numbers in the catalog come from official datasheets (Apple Support tech specs, NVIDIA `data-center/*`, AMD `instinct-tech-docs`); when adding a new preset cite the source in an inline comment so future updates can re-verify.

### TypeScript strictness

`tsconfig.app.json` and `tsconfig.node.json` enable `strict`, `noImplicitOverride`, `noImplicitReturns`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes`. Do not relax these flags — fix the underlying code instead. In particular:

- `field?: T` does NOT mean the field can be set to `undefined`. If a caller wants to pass `undefined` explicitly, the type must read `field?: T | undefined`.
- Array / object index access returns `T | undefined` — narrow with explicit guards (`if (!item) return;`) or `?? fallback`. Do not use non-null assertions (`!`).
- Avoid `any` and type casts. If a third-party type forces a cast, isolate it behind a typed helper.

## File Responsibilities

| File | What it does | What it must NOT do |
|---|---|---|
| `lib/calculator.ts` | Pure math functions only | No React, no fetch, no DOM |
| `lib/scoring.ts` | UI scoring utils: score normalization, color mapping, TPS labels | No React, no fetch |
| `lib/models.ts` | Static model catalog (KNOWN_MODELS, getModelGroups) | No formulas, no fetch |
| `lib/quants.ts` | Quantization constants: QUANT_SPECS (source of truth), QUANT_BITS, WEIGHT_QUANTS, KV_QUANTS, QUANT_FAMILY_ENGINES, getQuantFamily, getWeightQuantGroups | No model data, no logic |
| `lib/calcInput.ts` | `resolveModel` + assemble `CalcOptions` / `ValueScoreInput` from a `CardData` | No formulas, no fetch, no React |
| `lib/enginePresets.ts` | Engine preset catalog + `resolveActiveEngine` + `pickCompatibleEngine` (auto-snap) | No React, no fetch (must remain pure to be shared by UI and Footer) |
| `lib/hardwarePresets.ts` | Hardware preset catalog (Apple Silicon Max/Ultra, NVIDIA datacenter/consumer, AMD Instinct) + `resolveActiveHardware` + `PRESET_OWNED_FIELDS` allow-list | No React, no fetch (must remain pure — shared by `AvailableHardware` UI and Footer docs) |
| `lib/hf.ts` | Fetch HF config, detect formula/precision/activeParams | No React state, no routing |
| `lib/state.ts` | URL encode/decode only | No fetch, no React hooks |
| `lib/screenshot.ts` | PNG export via html-to-image | No React state, no formulas |
| `lib/types.ts` | Type definitions only | No logic |
| `hooks/useCalcResult.ts` | Memoized RAM calculation for a card config | No side effects, no fetch |
| `hooks/useValueScore.ts` | Memoized TPS / value score for a card config | No side effects, no fetch |
| `hooks/useHfModelImport.ts` | HF fetch flow with loading/error/warning state | No JSX, no URL state |
| `components/ConcurrentUsersInput.tsx` | Users + engine dropdowns; delegates "which preset is active" to `resolveActiveEngine` | No formulas; must NOT export non-component values (kills react-refresh) |
| `components/ThemeProvider.tsx` | Single `next-themes` wrapper with project defaults (`attribute="class"`, `defaultTheme="system"`, `storageKey="theme"`) | No JSX beyond the provider; do not import elsewhere — wrap `<App />` once in `main.tsx` |
| `components/ThemeToggle.tsx` | DropdownMenu Light / Dark / System, lives in `Header` | No theme logic of its own — must read/write through `useTheme()` only |

> **`components/ui/*`** — shadcn/ui primitives, configured via `components.json` (style `base-nova`, alias `@/components/ui`). To add a new primitive run `npx shadcn add <name>` — do NOT hand-write or hand-edit files in `ui/`, otherwise `npx shadcn diff` / future upgrades will silently overwrite your changes. If you really need a project-specific tweak, wrap the primitive in a sibling component (e.g. `InfoTooltip.tsx` wraps `ui/tooltip`).

### Icons

Single source for all icons in **our** code: [`react-icons`](https://react-icons.github.io/react-icons/).

- UI / outline icons → `react-icons/lu` (Lucide). Example: `import { LuCamera, LuChevronDown } from "react-icons/lu";`
- Brand / logo icons → `react-icons/si` (Simple Icons, 3000+ brands). Example: `import { SiGithub, SiHuggingface } from "react-icons/si";`
- Always pass `aria-hidden="true"` for purely decorative icons (the surrounding `<button>`/`<a>` already provides the accessible name via `aria-label` or visible text).
- Do NOT add `lucide-react` imports to files outside `src/components/ui/*`. `lucide-react` stays in `dependencies` only because shadcn/ui primitives import from it directly — the rest of the codebase uses `react-icons` so we keep one icon convention everywhere.
- Avoid inline `<svg>` markup in components — search `react-icons/{lu,si,md,fa,hi}` first; if nothing fits, add a tiny wrapper component in `src/components/icons/` rather than copy-pasting raw paths into JSX.
- The only **legitimate** inline `<svg>` in the codebase lives in `BrandIcon.tsx` (Microsoft / DeepSeek). They are intentionally absent from Simple Icons due to trademark restrictions, so we ship custom replicas. Brand colours for the rest are mirrored from the simple-icons palette and live next to `BRAND_ICONS` — that lets us keep `react-icons/si` as the single brand-icon source and avoid a separate `simple-icons` dependency.

### Design tokens

All colours go through CSS custom properties defined in `src/index.css`. Tailwind utilities such as `bg-success`, `text-info`, `border-cat-vision/30` resolve to these tokens at build-time via the `@theme inline {…}` mapping. Both light and dark themes provide values for every token — that is what makes the theme switcher work without per-component conditionals.

| Token group | Tokens | When to use |
|---|---|---|
| Surface | `background`, `foreground`, `card`, `popover`, `muted`, `secondary`, `accent`, `border`, `input`, `ring` | Default chrome — same as shadcn defaults |
| Brand | `primary`, `primary-foreground` | Emphasised actions, focus rings, brand chips |
| Status | `success`, `warning`, `danger`, `info` (+ `*-foreground`, `*-soft`) | Fits / tight / exceeds states; informational plates (Hosting, HF Import, hardware section). `*-soft` = tinted background for plates / chips, `*-foreground` = readable text on a soft background |
| Categorical (decorative) | `cat-vision`, `cat-reasoning`, `cat-tools` (+ `*-soft`) | Capability badges only |
| Chart series | `chart-1` … `chart-5` | Stacked / categorical data series in `ResultCard`, comparison views |
| Destructive | `destructive` | Existing shadcn slot — keep using it for destructive button hovers (Clear all) |

**Hard rule:** do NOT introduce hardcoded Tailwind palette classes (`bg-emerald-400`, `text-sky-300`, `border-violet-500/30`, …) in components. They look fine in dark mode and break in light. If a needed shade does not exist as a token, **add the token** in both `:root` and `.dark` plus the `@theme inline {…}` mapping; do not paper over with one-off colours.

### Theming

- The provider lives in `src/components/ThemeProvider.tsx` and is mounted exactly once around `<App />` in `src/main.tsx`.
- Storage key is `"theme"` and the value is one of `"light" | "dark" | "system"`. `index.html` ships an inline anti-flash bootstrap that reads the same key BEFORE first paint and applies `class="light"` / `class="dark"` to `<html>` so there is no flash of incorrect theme. Any change to the storage key must be mirrored in BOTH `ThemeProvider`'s `storageKey` AND the inline script in `index.html` — they are two halves of one contract.
- Every new colour must work in both themes. The fastest way to check: open the page, click the theme toggle, and look at the affected component. If a status chip becomes invisible on white, you used a token (`bg-success-soft`) without a matching `text-success-foreground` on the text — fix the contrast inside the token, not in the consumer.
- Add a new semantic token in three places: `:root { --foo: oklch(...) }`, `.dark { --foo: oklch(...) }`, and `@theme inline { --color-foo: var(--foo) }`. Tailwind autogenerates `bg-foo`, `text-foo`, `border-foo` after that — no extra config needed.

## Testing Conventions

- Tests live in `src/{lib,hooks,components}/__tests__/*.test.{ts,tsx}` (unit) and `e2e/*.spec.ts` (e2e). Use `.tsx` only when the test renders JSX.
- Unit tests use **Vitest** with `describe`/`it`/`expect`. Do not use Jest APIs.
- `src/test-setup.ts` wires `@testing-library/jest-dom` matchers — that gives you `toHaveTextContent`, `toBeInTheDocument`, etc. Do not register matchers per-file.
- Component tests use `@testing-library/react` (+ `@testing-library/user-event` for click flows). Hook tests use `renderHook` from the same package.
- Tests must find **real bugs**, not just document current behavior. If a test would pass even with a broken implementation, rewrite it with tighter assertions. When you hit an existing bug that you cannot fix in scope, document it with a `KNOWN QUIRK:` comment so the next change is conscious.
- Avoid mocking what you can compute directly. For `fetchHfConfig` tests, mock only `fetch` globally via `vi.stubGlobal("fetch", vi.fn())` and restore with `vi.unstubAllGlobals()` in `afterEach`. For `useHfModelImport` mock `fetchHfConfig` itself via `vi.mock("@/lib/hf", …)`.
- Do not use `any` or type casts unless absolutely necessary.
- E2E tests use `data-slot` attributes for stable locators on UI primitives (e.g. `[data-slot="combobox-trigger"]`, `[data-slot="select-item"]`, `[data-slot="card-content"]`). Use `getByPlaceholder` for the combobox search input since it has no `data-slot`.
- When a card contains **multiple instances of the same primitive** (e.g. several `Select`s — Weights Quant, KV Cache Quant, Concurrent Users, Inference Engine), do NOT address them by `nth()` — that breaks the moment a new control is added. Add a semantic `data-testid` to each trigger (e.g. `data-testid="weights-quant-trigger"`) and locate via `[data-testid="…"]`, optionally scoped inside `[data-slot="card-content"].nth(cardIndex)` for compare mode.

## Adding a New KV Formula

1. Add the new variant to the `KvFormula` union type in `src/lib/types.ts`
2. Add a `case` to the `switch (formula)` in `calcLLMRam` in `src/lib/calculator.ts`
3. Add the same `case` to the `switch (formula)` in `calcValueScore` in `src/lib/calculator.ts`
4. Add the formula label to `KV_FORMULA_LABELS` in `src/components/ResultCard.tsx`
5. Add the option to `FORMULA_OPTIONS` in `src/components/CustomModelForm.tsx`
6. Add formula detection logic to `detectFormula` in `src/lib/hf.ts` if it can be auto-detected from HF config
7. Write unit tests for the new formula in `src/lib/__tests__/calculator.test.ts`

## Adding a New Known Model

1. Add an entry to `KNOWN_MODELS` in `src/lib/models.ts`
2. Required fields: `displayName`, `brand`, `params`, `layers`, `kvHeads`, `headDim`, `moe`, `maxContextK`
3. Optional but important: `hfRepoId`, `kvFormula` (defaults to `"standard"`), `capabilities`
4. For hybrid: add `fullLayers`, `slidingWindow`, optionally `fullKvHeads`, `fullHeadDim`, `kvFactor`
5. For MLA: add `kvLoraRank`, `qkRopeHeadDim`
6. For linear_hybrid: add `fullLayers`
7. For MoE: set `moe: true` and `activeParams`. **`displayName` must follow the `<Brand> <Model> <Total>B-A<Active>B (MoE)` convention** (e.g. `"Qwen 3 235B-A22B (MoE)"`, `"DeepSeek V3 671B-A37B (MoE)"`). For models with established marketing names containing the expert grid (Mixtral `8x7B`, `8x22B`), keep that name and append `-A<Active>B` (e.g. `"Mixtral 8x7B-A13B (MoE)"`). Round to the nearest integer B.
8. Run `npm run test:unit` — no tests should break

## PR Checklist

- [ ] `npm run lint` passes with no new errors
- [ ] `npm run typecheck` passes (no relaxing of strict flags)
- [ ] `npm run test:unit` — all 200+ tests green
- [ ] `npm run test:e2e` — all 24+ tests green
- [ ] `npm run build` succeeds
- [ ] If a formula changed: both `calcLLMRam` and `calcValueScore` updated consistently
- [ ] If a new model added: `displayName`, `brand`, and `hfRepoId` are set; MoE follows the `<Total>B-A<Active>B (MoE)` naming convention
- [ ] If `engineId` / `kvCacheFillPct` touched: parent updates BOTH in one `setState`
- [ ] No hardcoded Tailwind palette classes (`bg-emerald-400`, `text-sky-300`, …) — use semantic tokens (`bg-success`, `text-info`)
- [ ] If a new component has theme-sensitive colours: visually verified in **both** Light and Dark
- [ ] No `any` types introduced, no non-null assertions (`!`)
- [ ] No `console.log` left in production code

CI mirrors this list as separate jobs in `.github/workflows/ci.yml` (lint, typecheck, unit + coverage, build, e2e). All must be green to merge.

---
> Source: [smelukov/WeightRoom](https://github.com/smelukov/WeightRoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
