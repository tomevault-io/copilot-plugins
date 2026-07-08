## erc8004-subgraph

> Reglas para desarrollo del subgraph ERC-8004 de Global Score Agent


# ERC-8004 Subgraph — Global Score Agent

## Stack

- **The Graph** (AssemblyScript), desplegado en **Ormi 0xGraph**
- **NO** es Envio HyperIndex — no usar `config.yaml` ni handlers TypeScript Envio

## Antes de editar

1. Leer [AGENTS.md](../AGENTS.md) y [docs/operaciones.md](../docs/operaciones.md)
2. Tras cambios en schema/mappings: `npm run codegen && npm run build`

## Deploy

- Un subgraph por chain; editar `network` + `startBlock` en `subgraph.yaml`
- Referencia: `networks.json` y `docs/propuesta-chains-erc8004.md`
- startBlock ≈ enero 2026, no genesis

## No tocar sin confirmación

- Contratos: `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`, `0x8004BAa17C55a88189AE136b182e5fdA19dE9b63`
- Firmas de eventos y formato de IDs (`{network}-{agentId}`)

---
> Source: [GlobalScoreAgent/global-score-indexer](https://github.com/GlobalScoreAgent/global-score-indexer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
