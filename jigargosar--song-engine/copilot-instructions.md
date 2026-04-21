## song-engine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A browser-based procedural music generator. Given a numeric seed, `generateSong(seed)` deterministically produces a complete multi-instrument song (pads, bass, drums, arp, melody) with chord progressions, energy curves, and BPM variations. Playback uses Tone.js. No framework — plain TypeScript + DOM.

## Commands

```bash
pnpm dev              # Vite dev server
pnpm build            # lint + vite build (production)
pnpm lint             # tsc --noEmit + biome check + eslint
pnpm typecheck        # tsc --noEmit only
pnpm lint:biome       # biome check src/
pnpm lint:eslint      # eslint src/
pnpm audit:coupling   # AST-based import/coupling analysis (ts-morph)
pnpm audit:tda        # Tell-Don't-Ask smell detection
pnpm audit:report     # Generate JSON audit report
```

No test framework is configured. No CI pipeline exists.

## Architecture

Four layers, strict dependency direction (lower layers never import higher):

**Layer 0 — Foundation** (no project deps):
`brand.ts` (nominal typing primitive), `NonEmpty.ts` (closure-based non-empty array)

**Layer 1 — Domain types & leaf generators** (depend on L0 only):
`rng.ts`, `note.ts`, `timing.ts`, `dynamics.ts`, `drums.ts`, `markov.ts`, `voice-leading.ts`, `arp.ts`, `melody.ts`

**Layer 2 — Orchestrator:**
`song.ts` — `generateSong(seed)` is pure, assembles all L1 generators into a `Song` data object

**Layer 3 — Impure output:**
`playback.ts` (Tone.js audio engine), `ui.ts` (DOM rendering)

**Layer 4 — App shell:**
`main.ts` (AppState discriminated union, event wiring), `global.css` (Tailwind v4)

### Pure/Impure Wall

`generateSong()` returns a plain data object with all event timing pre-computed as absolute seconds. No audio objects, no DOM references. `startPlayback(song, callbacks)` is the sole bridge to Tone.js — all 17 synth nodes live in its closure scope.

### Key Patterns

- **Branded primitives**: Every domain primitive (`Seed`, `Bpm`, `Seconds`, `NoteName`, `MidiNumber`, `ChordSymbol`, `RomanNumeral`) uses `Brand<T, B>` from `brand.ts`. Constructor functions share the type name: `Bpm(128)`, `NoteName("C4")`.
- **`as` casts banned**: ESLint rule `consistent-type-assertions: never` enforces zero `as` casts except the single suppressed line in `brand.ts`.
- **NonEmpty\<T\>**: Closure-based opaque type. Use `NonEmpty.init(arr)`, `NonEmpty.fromArray(arr)`, `NonEmpty.head(ne)`, `NonEmpty.toArray(ne)`. `Rng.pick()` requires `NonEmpty<T>` — can't pick from empty.
- **Explicit RNG threading**: No global RNG. Every random-consuming function takes `rng: Rng` as a parameter, preserving deterministic seed-based reproduction.
- **ISI (Make Impossible States Impossible)**: `AppState` is a tagged union — `stopped` has no song/controls, `playing`/`paused` require both. `DrumPattern` is a fixed-length 8-tuple. `Motif` internals are opaque.
- **Closure-as-encapsulation**: `startPlayback()` captures Tone.js nodes in closure scope; `stop()` disposes via closure — no module-level audio state.

## Strict TypeScript Config

- `noUncheckedIndexedAccess` — all `arr[i]` returns `T | undefined`, must be narrowed
- `exactOptionalPropertyTypes` — `undefined` must be explicit in optional props
- `noPropertyAccessFromIndexSignature` — use bracket notation for index signatures
- `noUnusedLocals` / `noUnusedParameters` — no dead code

## Lint Rules to Know

- **Biome**: `noExplicitAny: error`, `noNonNullAssertion: error`, `useConst: error`
- **ESLint**: `@typescript-eslint/consistent-type-assertions` with `assertionStyle: 'never'` — no `as` casts anywhere in `src/`

## Key Libraries

- **tonal** — music theory: `Scale`, `Progression`, `Voicing`, `VoicingDictionary`, `VoiceLeading`
- **tone** — Web Audio: synths, samplers, transport scheduling
- **seedrandom** — deterministic PRNG (wrapped by `rng.ts`)
- **remeda** — functional utilities (available but lightly used)
- **neverthrow** — Result type (available but not yet used in source)

## Design Documents

`docs/` contains architectural analysis — not user documentation:
- `walkthrough.md` — full codebase walkthrough with dependency diagram
- `extraction-plan.md` — refactoring plan from the original monolith
- `arch-review.md` — open architecture issues
- `audit-findings.md` — known bugs and design issues
- `coupling-audit-design.md` — design for the AST audit tooling in `scripts/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jigargosar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
