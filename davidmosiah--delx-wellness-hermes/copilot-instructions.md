## delx-wellness-hermes

> This repo packages the Delx Wellness Hermes profile, onboarding, skills and MCP connector presets.

# Agent Development Notes

## Scope

This repo packages the Delx Wellness Hermes profile, onboarding, skills and MCP connector presets.

## Commands

- Install: `npm ci`
- Typecheck: `npm run typecheck`
- Build: `npm run build`
- Test: `npm test`
- Dry-run install: `npm run install:dry-run`
- Doctor: `npm run doctor`
- E2E: `npm run e2e`

## Rules

- Never commit Hermes local profiles, model credentials, MCP tokens, OAuth secrets, or personal health data.
- Keep setup idempotent and safe to re-run.
- Keep Nourish as the no-OAuth smoke path.
- When connector behavior changes materially, update the Delx Wellness registry/docs surfaces too.

---
> Source: [davidmosiah/delx-wellness-hermes](https://github.com/davidmosiah/delx-wellness-hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
