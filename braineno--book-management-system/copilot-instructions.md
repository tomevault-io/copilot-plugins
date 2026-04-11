## book-management-system

> This repository builds a Windows-first, offline bookstore management system in Flutter desktop.

# AGENTS.md

## Project mission
This repository builds a Windows-first, offline bookstore management system in Flutter desktop.
Prioritize the first commercial closed loop:
product master data -> purchase receipt -> inventory ledger -> retail sale -> member operations -> reporting -> local backup.

## Architecture rules
- Organize code by business feature under `lib/features/<feature>/`.
- Inside each feature, keep clear layers: `presentation/`, `domain/`, `data/`.
- Domain layer must not import Flutter UI, Drift, Hive, or platform APIs.
- Presentation layer must not access DAOs, Drift tables, or database classes directly.
- Repository contracts live in `domain/`; implementations live in `data/`.
- Cross-feature calls should go through use cases or repository abstractions, not widget-to-widget shortcuts.

## Business invariants
- Product master data and stock quantities are separate concerns.
- Money is stored as integer fen/cents, never floating point.
- Discount values are stored as integer percentage/basis values, never floating point.
- Stock changes must come from posted documents or explicit inventory adjustments.
- Never mutate stock balance directly without ledger records.
- Posting, stock mutation, member balance mutation, and audit logging must be transactional where applicable.
- High-risk actions require permission checks and audit logs.

## Scope control
- Do not expand a P0 task into P1/P2 work unless explicitly requested.
- Prefer the smallest vertical slice that delivers real business value.
- Preserve existing working behavior unless the user explicitly asks for a refactor or replacement.
- Never “fix” analyzer output by deleting business logic or rolling features back.

## Delivery workflow
- For non-trivial tasks, read the relevant files/docs first and produce a short implementation plan.
- For behavior changes and bug fixes, write or update tests first when feasible.
- Keep diffs small and localized.
- Explain trade-offs and risks when the requirement is ambiguous.
- If a migration is needed, update schema, mappers, repositories, and migration notes together.

## Validation
Before finishing, run or explain the intended validation:
- `dart format .`
- `flutter analyze`
- targeted `flutter test` for changed modules
- if UI behavior changed, include a short manual verification checklist

## Style
- Prefer explicit code over clever code.
- Keep widgets thin; put business rules in use cases/services.
- Use Chinese-facing labels/text only in UI-facing files.
- Add comments only where intent is non-obvious.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrainEno)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BrainEno)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
