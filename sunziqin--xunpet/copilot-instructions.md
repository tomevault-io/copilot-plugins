## xunpet

> This repository is XunPet, a Windows-first Electron + TypeScript + Live2D AI desktop pet.

# XunPet Agent Guide

This repository is XunPet, a Windows-first Electron + TypeScript + Live2D AI desktop pet.

Before making non-trivial changes, read:

1. `PROJECT_GUIDELINES.md`
2. `LONG_TERM_MEMORY.md`
3. `docs/PROJECT_MAP.md`
4. `docs/MODULE_CONTRACTS.md`
5. `docs/RUNTIME_FLOWS.md`
6. `docs/MODULE_STATUS.md`
7. Relevant `docs/plans/*.md`
8. The actual source, tests, config defaults, and runtime logs if available

## Core Rules

- Use the current source code and `src/shared/contracts.ts` as the source of truth.
- Keep renderer windows thin. Renderer code must not directly access secrets, local model services, screenshots, OCR, SmolVLM, Bilibili protocol internals, or business databases.
- Put orchestration in Electron main/Core services.
- Keep preload APIs window-role specific.
- Console refresh, bootstrap, and status snapshots must be read-only.
- Do not silently switch backends or hide failures. Return clear errors.
- Do not commit API keys, cookies, tokens, generated data, runtime logs, local model weights, Python virtual environments, or private assets.
- Update `CHANGELOG.md` for user-visible or architectural changes.
- Update module docs when contracts, runtime flow, config fields, or public behavior change.

## Verification

Use these commands before publishing substantial changes:

```powershell
npm run typecheck
npm run build
npm test
npm run test:capabilities
npm run check:governance
```

## Open Source Notes

XunPet defaults toward OpenAI-compatible API mode for easier onboarding. Local ASR, TTS, vision, embedding, and live-filter models are optional and must be installed explicitly by the user. Do not add model weights or generated local environments to the repository.

---
> Source: [sunziqin/XunPet](https://github.com/sunziqin/XunPet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
