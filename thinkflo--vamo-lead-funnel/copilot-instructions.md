## vamo-lead-funnel

> Global project conventions for the Vamo Lead Intake Funnel


# Vamo Lead Intake Funnel — Global Rules

## Project Context
Multi-channel lead intake funnel for Vamo/HeatOS. Vue.js 3 + NestJS + MongoDB.
Photos are shared HeatOS platform assets anchored to buildings, not lead attachments.
API spec: Lead Creation v1.2.0. Stages: qualification and discovery.

## Standards
Full standards are in `docs/standards/`. Read before generating code:
- `docs/standards/code-principles.md` — SOLID, DRY, Clean Code, AI rules
- `docs/standards/typescript.md` — strict, no any, naming, imports
- `docs/standards/architecture.md` — data model, services, API design

## Universal Rules
- TypeScript strict mode. No `any`. Explicit return types on exports.
- Functions ≤ 20 lines. Parameters ≤ 3. Nesting ≤ 2 levels.
- No `console.log`. No commented-out code. No placeholder implementations.
- No magic numbers/strings — use named constants from @vamo/shared.
- No new npm dependencies without written justification.
- Enums: `as const` objects, never TypeScript `enum` keyword.
- Imports: @vamo/shared for all types, enums, schemas, constants.
- Package manager: pnpm. Not npm. Not yarn.
- Named exports only (except where framework requires default).

## Architecture Constraints (Non-Negotiable)
1. Three collections: leads, buildings, assets. Never embed buildings in leads.
2. Images belong to buildings, not leads. Asset service is building-scoped.
3. Building dedup via Geoapify place ID (primary), postalCode + street fallback.
4. GDPR erasure anonymizes lead contact data. Conditionally retains building/asset data.
5. Same Docker image, all environments. Behavior via env vars only.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thinkflo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thinkflo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
