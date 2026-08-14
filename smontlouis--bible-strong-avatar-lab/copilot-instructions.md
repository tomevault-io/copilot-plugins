## bible-strong-avatar-lab

> Bible Strong Avatar Lab is a standalone React 19 + Vite 8 application for procedural SVG avatars.

# AGENTS.md

## Project

Bible Strong Avatar Lab is a standalone React 19 + Vite 8 application for procedural SVG avatars.
Read `CONTEXT.md` before changing behavior or persistence semantics.

## Commands

```bash
pnpm dev
pnpm typecheck
pnpm test
pnpm build
pnpm check
```

## Engineering rules

- Use TypeScript in strict mode.
- Keep geometry, playback and document operations framework-independent.
- Preserve the split between durable React state and high-frequency Motion values.
- Do not use `useMemo`, `useCallback` or `memo`; React Compiler handles memoization.
- Add or update focused tests for domain behavior.
- Run `pnpm typecheck` and the relevant test file while working, then `pnpm check` before committing.
- `src/features/export/standaloneEngine.generated.ts` is generated. Update it with `pnpm engine`, never by hand.
- Keep English, French and Simplified Chinese copy synchronized across `src/i18n/index.ts` and `src/i18n/zh.ts`.
- Preserve project JSON compatibility only with the current pre-release schema unless a migration is explicitly requested.

## UI conventions

- Reuse components under `src/components/ui/` rather than adding one-off controls.
- Keep selection, hover, focus and destructive actions visually consistent across avatars, expressions and animations.
- Prefer transform/opacity animation over layout animation for continuous interactions.

---
> Source: [smontlouis/bible-strong-avatar-lab](https://github.com/smontlouis/bible-strong-avatar-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
