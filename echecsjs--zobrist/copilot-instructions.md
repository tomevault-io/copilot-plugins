## zobrist

> Agent guidance for the `@echecs/zobrist` package — Polyglot standard Zobrist

# AGENTS.md

Agent guidance for the `@echecs/zobrist` package — Polyglot standard Zobrist
hash keys for chess positions.

**See also:** [`REFERENCES.md`](REFERENCES.md), [`SPEC.md`](SPEC.md)

See the root `AGENTS.md` for workspace-wide conventions.

**Backlog:** tracked in
[GitHub Issues](https://github.com/echecsjs/zobrist/issues).

---

## Project Overview

Data-centric package. Exports the 781 Polyglot standard Zobrist hash keys as a
readonly `bigint` array plus typed convenience accessor functions. No runtime
dependencies — keep it that way.

Named exports: `KEYS` (the raw 781-value array), `piece()`, `turn()`,
`castling()`, `enPassant()`.

---

## Commands

```bash
pnpm build              # bundle TypeScript -> dist/ via tsdown
pnpm test               # run all tests once
pnpm lint               # ESLint + tsc type-check
pnpm format             # Prettier
pnpm lint && pnpm test && pnpm build   # full pre-PR check
```

---

## Architecture Notes

- **ESM-only** — the package ships only ESM. Do not add a CJS build.
- No runtime dependencies — keep it that way.
- Chess types in `src/types.ts` intentionally duplicate `@echecs/position` types
  to avoid circular dependency (position depends on zobrist, not vice versa).
- `src/keys.ts` contains only the raw constant array. Logic lives in
  `src/index.ts`.
- The `turn()` function returns `KEYS[780]` for white and `0n` for black,
  matching the Polyglot convention that the turn key is XORed only when white is
  to move.
- En passant: the package provides the raw key values. The consumer is
  responsible for applying the Polyglot rule that en passant is only included
  when an enemy pawn can actually capture.

---
> Source: [echecsjs/zobrist](https://github.com/echecsjs/zobrist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
