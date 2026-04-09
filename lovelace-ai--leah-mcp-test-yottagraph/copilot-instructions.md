## leah-mcp-test-yottagraph

> Elemental API client usage, schema discovery, entity search, API gotchas, MCP servers. Read when building features that fetch or display data from the Query Server.

# Elemental API — The Platform Data Source

**This app is built on the Lovelace platform.** The Query Server is the
primary data source — use it first for any data needs (entities, news,
filings, sentiment, relationships, events). Do NOT call external APIs
(e.g. sec.gov, Wikipedia) for data that the platform already provides.

The Elemental API provides access to the Lovelace Knowledge Graph through
the Query Server. Use it to search for entities, retrieve properties,
explore relationships, and analyze sentiment. New data sources are added
regularly — use the discovery-first pattern to find what's available.

## Skill Documentation

For full endpoint documentation, read the **elemental-api skill** in
`skills/elemental-api/`. Start with `SKILL.md`, then `overview.md`.
These files are copied from `@yottagraph-app/elemental-api-skill` during
`npm install` (postinstall step) — if the directory is empty, run
`npm install` first.

Key files:
- `entities.md` — entity search, details, and properties
- `find.md` — expression language for searching by type, property, and relationships
- `schema.md` — entity types (flavors), properties (PIDs), and schema endpoints
- `relationships.md` — entity connections
- `events.md` — events involving entities
- `articles.md` — news mentions and article content

## Client Usage

All API calls go through `useElementalClient()` from `@yottagraph-app/elemental-api/client`.
Auth tokens and base URL are configured automatically by the `elemental-client` plugin.

```typescript
import { useElementalClient } from '@yottagraph-app/elemental-api/client';

const client = useElementalClient();

const results = await client.getNEID({ entityName: 'Apple', maxResults: 5 });
const report = await client.getNamedEntityReport(results.neids[0]);
const schema = await client.getSchema();
```

Types are also imported from the client:

```typescript
import type { NamedEntityReport, GetNEIDResponse } from '@yottagraph-app/elemental-api/client';
```

## Discovery-First Pattern

The knowledge graph contains many entity types and properties, and new datasets
are added regularly (e.g. Edgar filings, financial data). Do NOT hardcode entity
types or property names. Instead, discover them at runtime:

1. **Get the schema** — `client.getSchema()` returns all entity types (flavors)
   and properties (PIDs) available in the system. See `schema.md`.

   The schema response contains:
   - **Flavors** (entity types): Company, Person, GovernmentOrg, etc.
     Each flavor has a numeric ID and a human-readable name.
   - **PIDs** (properties): name, country, industry, lei_code, etc.
     Each PID has a type (`data_str`, `data_int`, `data_nindex`, etc.).
   - Properties with type `data_nindex` are references to other entities —
     resolve them with another `getPropertyValues` call.

   Use flavor names in `findEntities()` expressions and PID names in
   `getPropertyValues()`.

2. **Search with expressions** — `client.findEntities()` uses a JSON expression
   language to search by type, property value, or relationship. See `find.md`.
3. **Get property values** — `client.getPropertyValues()` fetches property data
   for specific entities.

This pattern lets agents work with any dataset without needing hardcoded
knowledge of what's in the graph.

## API Gotchas

> **WARNING -- `getSchema()` response nesting**: The generated TypeScript types
> put `flavors` and `properties` at the top level, but the actual API response
> nests them under a `schema` key. Using `response.properties` directly will
> crash with `Cannot read properties of undefined`. Always use:

```typescript
const res = await client.getSchema();
const properties = res.schema?.properties ?? (res as any).properties ?? [];
const flavors = res.schema?.flavors ?? (res as any).flavors ?? [];
```

> **WARNING -- `getPropertyValues()` takes JSON-stringified arrays**: The `eids`
> and `pids` parameters must be JSON-encoded strings, NOT native arrays. The
> TypeScript type is `string`, not `string[]`. Passing a raw array will silently
> return no data.

```typescript
const values = await client.getPropertyValues({
  eids: JSON.stringify(['00416400910670863867']),
  pids: JSON.stringify(['name', 'country', 'industry']),
});
```

### `getLinkedEntities` only supports graph node types

`getLinkedEntities(neid, { entity_type: ['document'] })` will fail at
runtime with _"entity_type document not a valid graph node type"_. The
`/entities/{neid}/linked` endpoint only supports three entity types:
**person**, **organization**, **location**. Documents, filings, articles,
financial instruments, events, and all other types are excluded — even
though the schema shows relationships like `filed` connecting organizations
to documents.

To traverse relationships to non-graph-node types, use `getPropertyValues`
with the relationship PID instead. Relationship properties (`data_nindex`)
return linked entity IDs as values. Zero-pad the returned IDs to 20
characters to form valid NEIDs.

```typescript
const pidMap = await getPropertyPidMap(client);
const filedPid = pidMap.get('filed')!;
const res = await client.getPropertyValues({
    eids: JSON.stringify([orgNeid]),
    pids: JSON.stringify([filedPid]),
});
const docNeids = res.values.map((v) => String(v.value).padStart(20, '0'));
```

See the **cookbook** rule for a full "Get filings for a company" recipe.

### `getNEID()` vs `findEntities()` for entity search

- **`client.getNEID()`** -- simple single-entity lookup by name
  (`GET /entities/lookup`). Best for resolving one company/person name.
- **`client.findEntities()`** -- expression-based batch search
  (`POST /entities/search`). Best for filtered searches (by type, property,
  relationship). See `find.md` for the expression language.

## Error Handling

```typescript
try {
    const data = await client.getNEID({ entityName: '...' });
} catch (error) {
    console.error('API Error:', error);
    showError('Failed to load data. Please try again.');
}
```

Methods on `useElementalClient()` return data directly and throw on non-2xx
responses. For full `{ data, status, headers }` access, import the raw
functions instead:

```typescript
import { getArticle } from '@yottagraph-app/elemental-api/client';

const response = await getArticle(artid);
if (response.status === 404) { /* handle not found */ }
```

## Lovelace MCP Servers (Optional)

Four MCP servers **may** be configured in `.cursor/mcp.json` for interactive
data exploration. They are NOT required — the REST client
(`useElementalClient()`) and skill docs cover the same data and are the
primary path for building features.

**Before referencing MCP tools, check your available tool list.** If tools
like `elemental_get_schema` don't appear in your MCP server list, the
servers aren't connected — just use the REST client and skill docs instead.
Do not report this as a problem; it's a normal configuration state.

| Server | What it provides |
|---|---|
| `lovelace-elemental` | Knowledge Graph: entities, relationships, events, sentiment, schema discovery |
| `lovelace-stocks` | Stock/financial market data |
| `lovelace-wiki` | Wikipedia entity enrichment |
| `lovelace-polymarket` | Prediction market data |

### Setup

`.cursor/mcp.json` is auto-generated by `init-project.js`. If it's missing,
run `node init-project.js --local` to regenerate it. For provisioned projects,
the servers route through the Portal Gateway proxy (no credentials needed).
For local development without a gateway, the servers require an
`AUTH0_M2M_DEV_TOKEN` environment variable.

The MCP Explorer page (`pages/mcp.vue`) also connects to these servers
through the Portal Gateway for interactive exploration in the browser.

### When MCP Servers Are Available

If the MCP tools appear in your tool list, you can use them for interactive
exploration alongside the REST client:

| Tool | Purpose |
|---|---|
| `elemental_get_schema` | Discover entity types (flavors), properties, and relationships |
| `elemental_get_entity` | Look up entity by name or NEID; returns properties |
| `elemental_get_related` | Related entities with type/relationship filters |
| `elemental_get_relationships` | Relationship types and counts between two entities |
| `elemental_graph_neighborhood` | Most influential neighbors of an entity |
| `elemental_graph_sentiment` | Sentiment analysis from news articles |
| `elemental_get_events` | Events for an entity or by search query |
| `elemental_health` | Health check |

MCP is convenient for schema discovery and entity resolution during planning.
For building UI features, always use the REST client (`useElementalClient()`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Lovelace-AI)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
