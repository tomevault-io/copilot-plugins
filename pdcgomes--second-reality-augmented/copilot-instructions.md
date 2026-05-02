## core-runtime

> Vanilla JS conventions for shared core runtime modules


# Core Runtime Conventions

## Module Rules

- Vanilla ES modules only — zero dependencies, no frameworks, no build step required
- Must work identically when imported by the editor (via Vite) and by the player (via native ES modules)
- Export clean named exports or a default class — no side effects on import

## Timing

All time values are in seconds, sourced from `AudioContext.currentTime`. Never use `Date.now()` or `performance.now()`.

## Testing

Every core module must have accompanying unit tests in `src/__tests__/`. Keep pure logic testable without a real WebGL context — mock `gl` where needed.

## API Stability

These modules are shared across editor and player. Changing an export signature requires updating both consumers.

---
> Source: [pdcgomes/second-reality-augmented](https://github.com/pdcgomes/second-reality-augmented) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
