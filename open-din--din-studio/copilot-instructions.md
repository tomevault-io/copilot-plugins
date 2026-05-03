## din-studio

> 2. `project/ROUTE_CARD.json`

# AGENTS — din-studio

## LOAD ORDER

1. `AGENTS.md`
2. `project/ROUTE_CARD.json`
3. One matching file in `project/EDITOR_NODE_SLICES.json`, `project/MCP_TOOL_SLICES.json`, or `project/SURFACE_MANIFEST.json`
4. One matching file in `project/skills/`
5. The exact source file and exact test file

## ROUTE HERE WHEN

- The request changes editor nodes, shell workflows, launcher panels, asset flows, MCP, or code generation.
- The request changes `project/COVERAGE_MANIFEST.json` or `project/SURFACE_MANIFEST.json`.

## ROUTE AWAY WHEN

- Public API, patch schema, package exports, docs/components -> `react-din`
- Runtime, compiler, registry, migration, FFI, WASM -> `din-core`
- Workspace routing or automation -> `din-agents`

## ENTRY POINTS

- `project/ROUTE_CARD.json`
- `project/EDITOR_NODE_SLICES.json`
- `project/MCP_TOOL_SLICES.json`
- `project/SURFACE_MANIFEST.json`

## SKILL MAP

- Editor node work -> `project/skills/editor-node-change/SKILL.md`
- Surface or workflow change -> `project/skills/surface-flow-change/SKILL.md`
- MCP work -> `project/skills/mcp-target-change/SKILL.md`
- Manifest maintenance -> `project/skills/surface-manifest-update/SKILL.md`

## HARD RULES

- `din-studio` owns editor and MCP surfaces, not the public schema or runtime semantics.
- Use slice manifests before opening broad editor or MCP modules.
- Keep manifests, feature docs, and tests aligned.
- Escalate only for shared IDs, public patch surface, or round-trip contract changes.

## VALIDATION

- `npm run lint`
- `npm run typecheck`
- `npm run test`
- `npm run test:e2e`

---
> Source: [open-din/din-studio](https://github.com/open-din/din-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
