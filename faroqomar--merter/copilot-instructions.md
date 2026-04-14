## merter

> Rules for Convex data modeling and Next.js dashboard work


# Convex And Next.js Rules

## Authority

- `.TODO/Plan.md`
- `.cursor/PROJECT_CONTEXT.md`
- `dashboard/AGENTS.md`
- `dashboard/convex/_generated/ai/guidelines.md`

## Data Layer

- Model the planned tables: `posts`, `flows`, `catalogs`, and `settings`.
- Keep validators and indexes aligned with the real review, posts, channels, catalogs, and settings pages.
- Preserve data needed for retries, edits, and publishing audit trails.
- Follow the generated Convex guidance for validators, bounded queries, public vs internal function registration, and index usage.

## Status Rules

- Use explicit status transitions.
- `pending_review` is the default post state after AI and pricing are applied.
- Published records must keep publish result metadata.

## Dashboard Rules

- Match the App Router routes in `Plan.md`.
- Build operator-focused pages, not generic admin templates.
- The review page must support compare, edit, approve, and decline actions.

## Integration

- Keep webhook payloads, Convex mutations, and UI actions consistent.
- Store stable local paths for media and catalogs when the plan requires them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faroqomar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/faroqomar)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
