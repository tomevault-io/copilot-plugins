## saleslite

> You are working in a documentation-first repository.

# Claude Code Instructions

You are working in a documentation-first repository.

## Prime Directive
Use `/docs` and existing source files as the only source of truth. Do not invent APIs, fields, routes, database columns, commands, behavior, environment variables, or product requirements.

## Required Behavior
Before editing files:
1. Read `/docs/_facts.md`.
2. Read `/docs/execution-plan.md`.
3. Read any relevant docs for the current phase.
4. Inspect existing source files before making assumptions.
5. Produce a file-scope plan.

## File-Scope Plan Format
```markdown
## File Scope Plan
### Files Read
- ...

### Files Allowed to Modify
- ...

### Files Not Allowed to Modify
- ...

### Risk Notes
- ...
```

## Implementation Rules
- Make minimal changes required by the current phase.
- Do not refactor unrelated files.
- Do not change business logic unless the phase explicitly requires it.
- Do not change public API contracts unless documented.
- Do not change database schema unless the phase explicitly requires a migration.
- Keep commits phase-focused.

## Validation Required
After changes, report:
- Build result
- Test result
- Lint/typecheck result if applicable
- Files changed
- Docs updated
- Risks/follow-up
- `READY FOR NEXT PHASE` or `BLOCKED`

## Sales Lite Project Rules

### Bahasa Indonesia
All user-facing UI copy MUST use Bahasa Indonesia (Indonesian language). This includes labels, buttons, error messages, dialogs, placeholders, and tooltips.

### Flutter Mobile MVP
- Current priority is the Flutter mobile app for sales reps.
- **Flutter app location:** `sales_lite_app/` (not repo root).
- All Flutter commands (`flutter analyze`, `flutter test`, `flutter build`, `flutter run`) must be run from `sales_lite_app/`.
- Do not implement the manager dashboard during mobile MVP phases.
- Dashboard screens may be represented as placeholder routes only if routing requires it.
- Do not run `flutter create` until Phase 1 is approved.

### Do Not Invent
- Do not invent API endpoints, database columns, domain fields, or backend behavior.
- If an API contract is not documented in `docs/api-contract.md`, use mock data clearly marked as `// TEMP`.
- If a requirement is missing, write `UNKNOWN` and ask for clarification.

### Source of Truth Hierarchy
When docs conflict, the following order applies (highest priority first):
1. `docs/product-vision.md`
2. `docs/requirements.md`
3. `docs/architecture.md`
4. `docs/ui-design-system.md`
5. `docs/implementation-plan.md`
6. `docs/decisions.md`
7. `README.md`, `CLAUDE.md`, `COPILOT.md`
8. Codebase (once created)

If a conflict is found, stop and report it before proceeding.

### State Management
- Use `StatefulWidget` / local state by default.
- Do not introduce Riverpod, BLoC, GetX, Provider, or any state management package unless the project already uses it.
- If state management complexity grows, propose it with rationale first.

### External Packages
- Do not add external packages without explaining why.
- Prefer built-in Flutter APIs over third-party packages.
- If a package is needed, document the reasoning before adding it.

---
> Source: [malikilmulki/SalesLite](https://github.com/malikilmulki/SalesLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
