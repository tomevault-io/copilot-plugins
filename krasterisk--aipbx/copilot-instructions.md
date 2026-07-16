## aipbx-core

> Core aiPBX full-stack conventions for all agent work


# aiPBX Core Rules

Planning root: `.planning/` (covers frontend + backend sibling repo).

Before coding, read:
- `.planning/PROJECT.md` — context and agent rules
- `.planning/GAPS.md` — prioritized backlog
- `.planning/intel/RISKS.md` — before touching telephony/billing

Backend repo: `c:/Users/Professional/WebstormProjects/aiPBX_backend` (sibling).

## Definition of Done (every phase)

1. `npm run lint:ts` passes (both repos if changed)
2. `npm run test:unit` / `npm test` passes for affected modules
3. User-facing UI: i18n keys in `ru` + `en` minimum
4. API changes: backend DTO + frontend entity types updated together
5. Voice/telephony changes: manual test checklist noted in PR/plan
6. Update `.planning/STATE.md` after phase completion

## Scope discipline

- One GAP per active phase; no drive-by refactors
- Do not touch `ari/`, `billing/`, `accounting/` without explicit phase
- Max 1 product phase + 1 GTM task in parallel

## Deploy

Production deploy requires commit tag `[deploy all]` or `[deploy:1|2|3]`. Never SSH to prod directly.

---
> Source: [krasterisk/aiPBX](https://github.com/krasterisk/aiPBX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
