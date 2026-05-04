## genexus18mcp

> > Required repo skills before editing KB logic:

# GeneXus MCP Protocol Guide

> Required repo skills before editing KB logic:
>
> 1. [GeneXus MCP Mastery](./.gemini/skills/genexus-mastery/SKILL.md)
> 2. [GeneXus 18 Guidelines](./.gemini/skills/genexus18-guidelines/SKILL.md)

This repository is MCP-first. The official transport is MCP over stdio or HTTP at `/mcp`. The old `/api/command` endpoint is no longer part of the gateway surface.

## Correct MCP flow

1. Initialize the session with `initialize`.
2. Discover the live surface with `tools/list`, `resources/list`, `resources/templates/list`, `prompts/list`, and `completion/complete` when needed.
3. Execute work with `tools/call`, `resources/read`, and `prompts/get`.
4. For HTTP MCP, send `MCP-Protocol-Version: 2025-06-18` and reuse the returned `MCP-Session-Id`.

## Recommended tool usage

- `genexus_query`: find objects, references, signatures, and dependency entry points. Supports optional `typeFilter` and `domainFilter` for server-side narrowing.
- `genexus_read`: read object parts with pagination. Pass singular `name` for one object or `targets[]` for coordinated multi-object reads (mutually exclusive). For MCP clients, keep reads paginated; the server intentionally returns a source-first first page when no explicit `offset`/`limit` is provided. For XML metadata parts such as `Layout`, `WebForm`, and `PatternInstance`, the gateway applies a larger metadata budget to avoid truncating the editable XML.
- `genexus_edit`: apply focused edits to a part or replace content through the MCP write path. Three modes:
  - `mode: 'xml'` (default): full XML replacement.
  - `mode: 'ops'`: typed semantic op catalog (`set_attribute`, `add_attribute`, `remove_attribute`, `add_rule`, `remove_rule`, `set_property`).
  - `mode: 'patch'`: JSON-Patch (RFC 6902) array over the canonical JSON object representation; legacy string-form text patch is also accepted for backward compatibility.
  Pass `targets[]` (array of edit-request objects) for atomic multi-object edits — mutually exclusive with singular `name`.
  `PatternInstance` targets the authoritative WorkWithPlus instance when the requested object is a WebPanel managed by WorkWithPlus.
- All write tools (`genexus_edit`, `genexus_create_object`, `genexus_refactor`, `genexus_forge`, `genexus_import_object`) accept:
  - `dryRun: true` — returns a preview envelope `{_meta:{dryRun,schemaVersion}, plan:{touchedObjects, xmlDiff, brokenRefs, warnings}}` without mutating the KB.
  - `idempotencyKey: '<token>'` (`[A-Za-z0-9_-]{1,128}`) — deduplicates retries; concurrent calls with the same key are coalesced; cache is per-KB, sliding TTL 15 min default. `dryRun` bypasses the cache.
- `genexus_inspect`: get structured conversion or object context.
- `genexus_analyze`: navigation, lint, UI, and summary analysis modes.
- `genexus_lifecycle`: build, validate, index, and KB lifecycle operations.
- `genexus_get_sql`: extract DDL and SQL-oriented schema insights.
- `genexus_create_object`: create new KB objects.
- `genexus_refactor`: supported rename and extraction refactors.
- `genexus_add_variable`: add variables through the worker contract.
- `genexus_format`: format source through the worker formatter.
- `genexus_properties`: get or set object properties.
- `genexus_asset`: find, read, and write KB assets such as `.xlsx` templates. Reads are metadata-first; request `includeContent=true` only for files small enough to carry as Base64 safely.
- `genexus_history`: list, read, save, and restore object history.
- `genexus_structure`: read or update logical and visual structure.
- `genexus_doc`: access documentation, health, and visualization flows.

## Resource-first patterns

Prefer resources when the data is naturally browsable or cacheable:

- `genexus://objects/{name}/part/{part}`
- `genexus://objects/{name}/variables`
- `genexus://objects/{name}/navigation`
- `genexus://objects/{name}/summary`
- `genexus://objects/{name}/indexes`
- `genexus://objects/{name}/logic-structure`
- `genexus://attributes/{name}`
- `genexus://kb/index-status`
- `genexus://kb/health`

## Operating rules

- Do not design new features around non-MCP transport contracts.
- Do not use retired tool names such as `genexus_patch`, `genexus_read_source`, `genexus_write_object`, `genexus_batch_read`, or `genexus_batch_edit`. The latter two were removed in v2.0.0 — use `genexus_read`/`genexus_edit` with `targets[]`. Calls to removed tools return JSON-RPC `-32601` with `error.data.replacedBy` and `error.data.argHint`; `initialize` advertises `_meta.removedTools` for proactive detection.
- Do not pass the legacy `changes` argument to `genexus_edit` — it was removed in v2.0.0; use `targets[]` instead.
- Read first, then edit. Use paginated reads for large objects.
- Prefer MCP discovery over hardcoded assumptions about available tools or resources.
- After C# changes, run `.\build.ps1`. For current validation commands, follow the README.

---
> Source: [lennix1337/Genexus18MCP](https://github.com/lennix1337/Genexus18MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
