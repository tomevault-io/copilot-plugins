## global-score-indexer

> Este documento orienta a agentes autónomos (Cursor, CI bots, etc.) que trabajen en el repositorio **global-score-indexer**.

# Guía para agentes de IA

Este documento orienta a agentes autónomos (Cursor, CI bots, etc.) que trabajen en el repositorio **global-score-indexer**.

## Fuente de verdad

- Este repo es un **subgraph The Graph** (AssemblyScript), **no** un indexer Envio.
- **ERC-8004 en producción:** subgraphs públicos **Agent0** en The Graph Network — import vía Supabase `graphs.*`. Ver [`docs/agent0-gsa-field-mapping.md`](docs/agent0-gsa-field-mapping.md) y [`docs/arquitectura-grafos-gsa.md`](docs/arquitectura-grafos-gsa.md).
- **Subgraphs propios** (Ethos, Virtual, ERC-8183): despliegue en **Goldsky**.
- **Olas Mech Marketplace:** subgraph **oficial Autonolas** — `https://api.subgraph.autonolas.tech/api/proxy/marketplace-{base|gnosis}`.
- **Ormi 0xGraph retirado** (jul 2026) — no documentar como proveedor activo ni desplegar ahí.
- Metadatos por chain: [`networks.json`](networks.json) (ERC-8004 referencia), [`networks-olas.json`](networks-olas.json), [`networks-virtual.json`](networks-virtual.json), [`networks-8183.json`](networks-8183.json), [`networks-ethos.json`](networks-ethos.json).
- Especificación Olas: [`docs/olas-marketplace-indexer.md`](docs/olas-marketplace-indexer.md).
- Especificación Virtual: [`docs/virtual-marketplace-indexer.md`](docs/virtual-marketplace-indexer.md).
- Especificación ERC-8183: [`docs/erc-8183-commerce-indexer.md`](docs/erc-8183-commerce-indexer.md).
- Backlog de chains: [`docs/propuesta-chains-erc8004.md`](docs/propuesta-chains-erc8004.md).

## Reglas estrictas

**No modificar sin confirmación del usuario:**

- Direcciones de contrato (`0x8004A169...`, `0x8004BAa1...`)
- Firmas de eventos en manifests Olas o ERC-8004 (`subgraph.yaml` raíz)
- Formato de IDs en handlers (`{network}-{agentId}`, etc.)

**Flujo obligatorio** tras cambiar `schema.graphql`, manifests Olas o `subgraphs/olas-marketplace/src/mapping.ts`:

```bash
npm run codegen:olas-base   # o codegen:olas-gnosis
npm run build:olas-base     # o build:olas-gnosis
```

Tras cambiar `subgraphs/virtual-marketplace/`:

```bash
npm run codegen:virtual-base
npm run build:virtual-base
```

Tras cambiar `subgraphs/erc-8183-commerce/`:

```bash
npm run codegen:8183-base   # o codegen:8183-bsc
npm run build:8183-base     # o build:8183-bsc
```

Tras cambiar `subgraphs/ethos-network/`:

```bash
npm run codegen:ethos-base
npm run build:ethos-base
```

Para ERC-8004 (raíz, referencia local): `npm run codegen && npm run build`.

**Modelo de deploy:**

| Producto | Prod | Repo (referencia / Goldsky) |
|----------|------|------------------------------|
| ERC-8004 HUMI | **Agent0** (no deploy desde repo) | Código raíz para local dev |
| Olas, Virtual, Ethos, 8183 | **Goldsky** | `subgraphs/*` + `npm run deploy:*-goldsky` |

**Start block:** bloque ~enero 2026 (cuando ERC-8004 se desplegó en esa red), no genesis. Ver [docs/operaciones.md](docs/operaciones.md).

## Archivos clave

| Archivo | Propósito |
|---------|-----------|
| `subgraph.yaml` | Manifest ERC-8004 (raíz): referencia / local dev |
| `subgraphs/olas-marketplace/` | Referencia Olas — **prod = Autonolas oficial** |
| `subgraphs/virtual-marketplace/` | Virtual Base → Goldsky |
| `subgraphs/erc-8183-commerce/` | ERC-8183 Base + BSC → Goldsky |
| `subgraphs/ethos-network/` | Ethos Base → Goldsky |
| `networks*.json` | Contratos, startBlocks, endpoints |
| `docs/arquitectura-grafos-gsa.md` | Arquitectura maestro grafos |
| `docs/agent0-gsa-field-mapping.md` | Agent0 → normalize Supabase |

## Estado de productos (jul 2026)

| Producto | Import prod | Deploy subgraph |
|----------|-------------|-----------------|
| ERC-8004 (5 chains) | Agent0 | Mantenido por agent0lab |
| Olas Base/Gnosis | **Autonolas** oficial | Mantenido por Valory |
| Ethos Base | Pendiente seed `graphs.*` | Goldsky `ethos-network-base/prod` |
| Virtual Base | `graphs.*` | Goldsky `virtual-acp-base/prod` |
| ERC-8183 Base/BSC | `graphs.*` | Goldsky `erc-8183-commerce-*` |

## Procedimientos

- Deploy y checklist: [docs/operaciones.md](docs/operaciones.md)
- Contexto de negocio: [docs/negocio.md](docs/negocio.md)
- Arquitectura grafos: [docs/arquitectura-grafos-gsa.md](docs/arquitectura-grafos-gsa.md)

## Errores comunes

- Asumir que hay que `graph deploy` a Ormi para ERC-8004 prod — **incorrecto**; prod = Agent0 en Supabase.
- Olvidar `receipt: true` en el handler `NewFeedback` (necesario para `gasUsed` en código repo).
- IDs inconsistentes entre `Agent`, `Feedback` y `FeedbackResponse`.
- Desplegar subgraph propio sin tag `prod` en Goldsky.
- Usar startBlock posterior al primer evento (pérdida de datos).

---
> Source: [GlobalScoreAgent/global-score-indexer](https://github.com/GlobalScoreAgent/global-score-indexer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
