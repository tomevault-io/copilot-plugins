## ai-automation

> - Repository status: bootstrap only

# Repository Memory

## Current State
- Repository status: bootstrap only
- Current tracked content: Git metadata only
- Application source code: not present yet
- Documentation status: initial project memory files created on 2026-05-08

## Working Rules For Future Agents
- Do not assume frontend, backend, database, or deployment architecture exists until source files are added.
- Update the memory files in `docs/` whenever new modules, APIs, schema objects, or operational flows are introduced.
- Keep documentation factual and derived from the current repository state.
- If implementation and documentation diverge, treat source code as the primary truth and refresh these files.

## Engineering Standards
- Follow the senior NestJS + NextJS standards defined by the user.
- Prioritize readability, consistency, separation of concerns, reusability, and predictability.
- Keep business logic out of controllers, pages, and large UI components.
- Add English JSDoc and comments where required by project policy.
- Avoid inline styles in frontend code.

## Memory File Index
- `docs/SYSTEM_OVERVIEW.md`: high-level architecture and system boundaries
- `docs/FEATURE_MAP.md`: implemented features and feature ownership map
- `docs/DATABASE_SCHEMA.md`: database entities, relationships, and migration notes
- `docs/API_MAP.md`: endpoint inventory and contract notes
- `docs/BUG_FIX_PLAYBOOK.md`: debugging workflow and safe change process

## Current Reality Check
- No application runtime, package manifest, or framework configuration has been added yet.
- No database schema, API surface, or user-facing feature set exists in the repository at this time.
- All memory docs currently describe the empty bootstrap state and define how they should be maintained as the project grows.

---
> Source: [nhatphat112/ai-automation](https://github.com/nhatphat112/ai-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
