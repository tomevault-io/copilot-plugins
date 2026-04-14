## life-system-shared

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`@life-system/shared` — shared TypeScript schemas and types used across the Life System (between The Ansible backend and Jane AI agent). This is a read-only standalone extraction from the main monorepo.

## Build Commands

```bash
npm run build        # Compile TypeScript → dist/
npm run typecheck    # Type-check only (no emit)
npm run dev          # Watch mode
```

There are no tests or linter configured in this repo.

## Architecture

**Zod schemas are the single source of truth.** All TypeScript types are derived via `z.infer<>` — never define types manually.

The codebase is four files in `src/`:

- **`schemas.ts`** — Zod runtime validation schemas for all data structures. Read this file to understand the data model (the generated `.d.ts` files expand Zod internals and are hard to read).
- **`types.ts`** — TypeScript types inferred from schemas. Never edit types directly; change the schema instead.
- **`constants.ts`** — Shared constants and category/status enums using `as const` pattern.
- **`index.ts`** — Barrel export. Also re-exports `uuidv7` from the dependency.

## Key Domain Concepts

- **CommunicationEvent**: Core message format flowing through NATS between channels (Slack, email, canvas, Claude Code). Always Markdown content. Uses UUIDv7 for time-sortable IDs. `communicationEventInputSchema` is for creation (optional `id`), `communicationEventSchema` is for validation of complete events.
- **Record**: Categorized data items (`people | projects | ideas | admin`) with confidence scores, tags, and flexible `data` payloads. `recordCreateSchema` omits server-generated fields.
- **InboxLog**: Audit trail for captured items with filing status.
- **JaneCommand / JaneExecutionView**: AI command execution tracking with cost/token accounting.
- **Confidence thresholds**: `>= 0.85` auto-files, `< 0.6` needs human review (see `CONFIDENCE_THRESHOLDS` in constants).

## Conventions

- ES modules (`"type": "module"`) — use `.js` extensions in imports even for TypeScript files
- Target: ES2022 with NodeNext module resolution
- Strict TypeScript enabled
- Schemas use `.passthrough()` for extensibility
- UUIDv7 (RFC 9562) for all IDs — time-sortable and globally unique

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/the-ansible)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/the-ansible)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
