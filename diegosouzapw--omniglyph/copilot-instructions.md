## omniglyph

> Context-as-image compression proxy for LLMs. Rewrites the bulky parts of each

# omniglyph — Agent Guidelines

## Project

Context-as-image compression proxy for LLMs. Rewrites the bulky parts of each
request (system prompt, tool docs, old history, large tool outputs) into dense
1-bit PNG pages before they leave the machine, using **exact per-provider
billing math** so conversion happens only when it is profitable. Ships as a
standalone proxy (Node + Cloudflare Workers) and as the `omniglyph` compression
engine inside [OmniRoute](https://github.com/diegosouzapw/OmniRoute).

## Stack

TypeScript (strict, ESM) · Vitest · esbuild (`scripts/build.mjs`) · tsx ·
Cloudflare Wrangler · pnpm. Node ≥18.

## Build / Test Commands

```bash
pnpm install && pnpm run build      # bundle to dist/
pnpm run typecheck                  # tsc --noEmit — must be clean
pnpm test                           # vitest run (full suite; slow — background it)
npx vitest run tests/<file>.test.ts # focused run while iterating
```

## Architecture

- `src/core/transform.ts` — the pipeline: pick bulky blocks, gate on
  profitability + model approval, reflow text, render, splice pages back into
  the request (cache-friendly ordering).
- `src/core/anthropic-vision.ts` — exact Anthropic image billing: 28px patches
  + per-block overhead, per-tier resize caps, page geometry. Both tiers render
  the standard 1568×728 page (the high-res tier is a measured billing trap).
- `src/core/gemini-model-profiles.ts` — Gemini billing formulas (tiles +
  `media_resolution`) and tile-native page geometry.
- `src/core/render*.ts` + `assets/` — 1-bit 5×8 glyph atlas rendering to PNG;
  deterministic renders are memoized (`render-cache.ts`, LRU).
- `src/core/proxy.ts` — request handling, refusal-retry sniffer (SSE/JSON
  replay with `retryRefusalWithOriginal` kill switch), savings accounting
  against a `count_tokens` counterfactual.
- `src/node.ts` — local host: dashboard, events file, upstream routing.
  `src/worker.ts` — Cloudflare Workers host.
- `src/core/index.ts` — public package API; billing math (`anthropicImageTokens`,
  `resolveAnthropicVisionTier`, tier caps) is re-exported at the root because
  OmniRoute's image-aware token estimator consumes it.
- `benchmarks/billing-sweep/` — proves the billing formulas (residual zero).
  `benchmarks/density-frontier/` — read-accuracy frontier per model/density;
  transports: direct API, OpenRouter, `--via-cli`, `--via-omniroute`.

## Conventions & Review Focus

- **Strict TDD** — failing test first, always. **Measurement before claims** —
  every published number needs a JSONL receipt in `benchmarks/*/results/`.
- Fail-closed gates are load-bearing: a change that lets an unapproved model
  receive images, or converts when the token math loses, is a defect even if
  tests pass.
- Never commit secrets; never hand-edit benchmark results; never weaken tests
  to go green.
- LICENSE keeps the upstream copyright line (MIT obligation); everywhere else
  the brand is OmniGlyph only — enforced by the rebrand guard in
  `tests/docs-integrity.test.ts`.
- No AI attribution trailers in commits/PRs/CHANGELOG.

## Reference Documentation

| doc | what |
|---|---|
| `docs/architecture/ARCHITECTURE.md` | one-page codebase map |
| `docs/benchmarks/BENCHMARKS.md` | methodology + every result table |
| `docs/ROADMAP.md` | work queue and measured-decision log |
| `benchmarks/README.md` | how the harnesses prove the savings |

---
> Source: [diegosouzapw/OmniGlyph](https://github.com/diegosouzapw/OmniGlyph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
