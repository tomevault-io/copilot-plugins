## omp-plugin-dynamic-system-prompt

> - Verify the three-axis resolver against the live catalog with `bun -e` sweep; no null-version among classified.

# AGENTS.md

## Build and test

- Verify the three-axis resolver against the live catalog with `bun -e` sweep; no null-version among classified.
- Use `bun` as the runner; do not use `npm` or `yarn` lockfiles.

## Tooling constraints

- Do not add runtime `dependencies`. Every import must be a Node builtin or an `omp` package resolved from the host; declare each host package imported for its values as a `peerDependency` with an open lower bound and no upper bound, so any omp at or above the floor satisfies it.
- Keep `skipLibCheck: true` in `tsconfig.json`; the host `pi-catalog` types require it.

## Plugin contract

- Preserve append-only behavior: never replace `omp`'s system prompt, only append `## Model tuning` via `before_agent_start`.
- Surface UI state through `ctx.ui.setStatus(STATUS_KEY, text)`; `ctx.ui.setHeader`/`setFooter` are no-ops in every shipped omp UI context. omp runs status text through `sanitizeStatusText`, so it must be plain single-line text — symbols, never ANSI colour.
- Check `event.systemPrompt.some(entry => entry.includes(TUNING_HEADER))` before appending; do not accumulate across turns.
- Resolve model identity via `@oh-my-pi/pi-catalog/identity` (`bareModelId`, `parseOpenAIModel`/`parseAnthropicModel`/`parseGeminiModel`/`parseGlmModel`, family predicates); never re-implement parsers.
- Normalize model ids by lowercasing `bareModelId(id)` and stripping `:thinking`/`:free`/`-cheaper`/`@default`/`[1m]`/`-high` etc. before classification.
- Derive `weight` from `FRONTIER_VERSION`/`GENERATION_STEP` plus `small` tier penalty; `unknown` must land `heavy`.
- Keep one rule per `concern`; sort by `priority` then `id` and trim from the end to `CHAR_BUDGET[weight]`.
- Read settings from `~/.omp/plugins/omp-plugins.lock.json` under `omp-plugin-dynamic-system-prompt`, falling back to `OMP_MODEL_TUNING*` env and manifest defaults; never throw on missing lockfile.
- A rule earns its place on one of two sources: a preset in the reference corpus, or first-party vendor prompting guidance — and in both cases only when omp's base prompt does not already state the same correction.
- Bound every `light` rule to the series its evidence names: exact version, a closed `band(...)`, or no version claim at all. `version >= X` with no ceiling is reserved for `medium`/`heavy` rules and for rules stating a harness fact rather than a model defect.

---
> Source: [metaphorics/omp-plugin-dynamic-system-prompt](https://github.com/metaphorics/omp-plugin-dynamic-system-prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
