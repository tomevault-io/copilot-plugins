## domain-agents

> `domain-agents` is a CLI tool that discovers business domains in TypeScript/Node codebases and generates AI agent files for evolutionary architecture. It uses static analysis (TypeScript compiler API, import graph clustering, naming patterns, dependency mapping) to identify domains, then generates per-domain agent markdown and a system map (AGENTS.md).

# CLAUDE.md

## Project Overview

`domain-agents` is a CLI tool that discovers business domains in TypeScript/Node codebases and generates AI agent files for evolutionary architecture. It uses static analysis (TypeScript compiler API, import graph clustering, naming patterns, dependency mapping) to identify domains, then generates per-domain agent markdown and a system map (AGENTS.md).

## Architecture

```
src/
  types.ts                 Core type definitions for the entire system
  discovery/
    structure.ts           Pass 1: directory analysis, detects feature/layer/mixed patterns
    imports.ts             Pass 2: TypeScript compiler API, builds import graph with export/import resolution
    naming.ts              Pass 3: clusters files by prefix/suffix/directory/keyword patterns
    dependencies.ts        Pass 4: maps npm packages to domains (stripe→billing, sendgrid→email, etc.)
    interfaces.ts          Pass 5: detects cross-domain interface points, calculates coupling scores
    index.ts               Orchestrator: combines all 5 signals, merges leaf clusters, handles hotspots
  generator/
    agent-file.ts          Generates per-domain markdown (purpose, interfaces, scaling, tech debt, observability)
    agents-md.ts           Generates AGENTS.md system map with dependency graph and cross-domain contracts
  hooks/
    lookup.ts              File-to-domain lookup from proposal.json (powers all integrations)
    cursor.ts              Generates .cursor/rules/*.mdc files with glob-based activation for Cursor
  mcp/
    server.ts              MCP server: exposes domain_lookup, list_domains, domain_context, domain_files tools
  generator/
    enrich.ts              LLM enrichment: reads domain code, calls Claude to generate contextual agent files
  cli/
    index.ts               CLI entry point (commander): discover, init, setup, health, hooks commands
    discover.ts            Runs discovery engine, saves proposal.json
    init.ts                Reads proposal, generates agents/*.md + AGENTS.md (with optional --enrich)
    setup.ts               Saves Anthropic API key to ~/.config/domain-agents/config.json
    health.ts              Checks boundaries, coupling, staleness against current codebase state
```

## Commands

```bash
npm test              # Run all tests (vitest)
npm run test:watch    # Watch mode
npm run build         # TypeScript compilation (tsc)
npx vitest run tests/discovery/    # Run only discovery tests
npx vitest run tests/cli/          # Run only CLI tests
npx vitest run tests/hooks/        # Run only hooks tests
```

## Key Design Decisions

- **Merge strategy**: Only leaf singletons (no outgoing imports, exactly 1 consumer domain) get auto-merged during clustering. This prevents aggressive cascading that swallows unrelated domains.
- **Hotspot detection**: Singletons importing from 3+ domains go to unassigned — they're coupling hotspots requiring human review.
- **Layer-organized support**: Uses naming prefix extraction (`auth.controller.ts` → "auth") to cluster files across technical layers (controllers/, services/, models/) into business domains.
- **Type-only imports**: Weighted at 0.3 (vs 1.0 for value imports) in the import graph, since type dependencies are weaker coupling signals.

## Test Fixtures

Three realistic TypeScript codebases in `tests/fixtures/`:
- `feature-organized/` — 17 files, clean directory-per-feature SaaS (the easy case)
- `layer-organized/` — 20 files, traditional MVC Express app (the hard case: must cluster across layers)
- `mixed/` — 17 files, partially organized with coupling hotspots (the real-world case)

Each fixture has real TypeScript files with actual import relationships — the import graph analysis depends on these being valid.

## Testing Approach

Strict TDD. Tests were written first against the 3 fixture codebases. The discovery engine must correctly handle all three organization patterns. Tests verify specific domain assignments, confidence ranges, coupling scores, and boundary detection.

## Style

- TypeScript with strict mode, ESM modules (Node16 resolution)
- No unnecessary abstractions — the codebase is straightforward and direct
- Each discovery pass is a standalone function that can be tested independently
- The orchestrator (`discovery/index.ts`) combines signals without over-coupling to individual passes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonnonz1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
