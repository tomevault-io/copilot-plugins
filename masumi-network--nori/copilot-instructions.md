## nori

> This repository contains the independent TypeScript Pi-agent runtime for Nori.

# Nori Agent Runtime Notes

This repository contains the independent TypeScript Pi-agent runtime for Nori.

## Active Agent

- Nori: DevRel for the Masumi/Sokosumi/Kodosumi ecosystem.

Do not add other coworkers or legacy Python/Azure runtime paths back into this repository. Other agents live in their own repositories.

## Runtime

- App: `apps/coworkers-core`
- Prompts: `src/agents/nori`
- Sokosumi Pi extension: `@masumi-network/pi-sokosumi`
- Pi package registration: `.pi/settings.json`
- Build/test: `pnpm typecheck`, `pnpm test`, `pnpm build`
- Start: `pnpm start`
- Health: `GET /healthz`

`@masumi-network/pi-sokosumi` is consumed from the separate GitHub repository. Keep imports through package subpaths like `@masumi-network/pi-sokosumi/worker`.

---
> Source: [masumi-network/nori](https://github.com/masumi-network/nori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
