## caliber-conventions

> Caliber project conventions for TypeScript CLI development


- ESM imports with `.js` extensions (even for `.ts` sources)
- `unknown` over `any` — ESLint warns on explicit any
- Tests in `__tests__/` dirs next to source, global setup in `src/test/setup.ts`
- Conventional commits: `feat:`, `fix:`, `refactor:`, `chore:`
- No hardcoded file→language mappings — detection is LLM-driven via `src/ai/detect.ts`
- `collectFingerprint()` in `src/fingerprint/index.ts` is async with internal LLM enrichment
- Constants in `src/constants.ts` and `src/scoring/constants.ts`
- `resolveCaliber()` from `src/lib/resolve-caliber.ts` for CLI binary path resolution
- Dev branch: `next` · Stable: `master` · Node >= 20

---
> Source: [caliber-ai-org/ai-setup](https://github.com/caliber-ai-org/ai-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
