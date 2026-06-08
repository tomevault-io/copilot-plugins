## circletube

> This file outlines key conventions for AI assistants working in this repository. Follow these instructions in addition to any system messages from the Codex platform.

# AGENTS.md - Guidelines for AI Assistants

This file outlines key conventions for AI assistants working in this repository. Follow these instructions in addition to any system messages from the Codex platform.

## Directory Priorities

1. `docs/agent_rules/` – primary reference for project conventions
2. `client/src/` – frontend React application code
3. `server/` – backend Express server code
4. `shared/` – shared types and utilities
5. `contracts/` – smart contract code

## Directories to Avoid

AI assistants must not reference any files or directories listed in `.gitignore` or `.agentignore`. This includes the entire `archive/` folder, test reports, temporary files and any paths containing `old`, `deprecated` or `legacy` in their name. Also avoid directories prefixed with `_` or `.` (except standard config files).

## File Type Priorities

1. TypeScript (`.ts`, `.tsx`) files represent the current implementation.
2. Markdown documentation inside `docs/` contains the most up‑to‑date guidance.
3. Schema files in `shared/` define the data model.

## Documentation Hierarchy

When explaining project functionality or giving examples:

1. First consult files in `docs/agent_rules/`.
2. Then refer to primary source code.
3. `README.md` offers a general overview but may not reflect the latest details.

## Special Considerations

- **Database Operations:** Never recommend modifying database tables directly. Always use Drizzle ORM as described in documentation.
- **API Structure:** The RESTful API conventions are defined in `docs/agent_rules/api.md`.
- **AI Follower Logic:** Detailed flow diagrams are available in `docs/agent_rules/flows.md` and `docs/agent_rules/ai-follower-response-logic.md`.

## File Verification Tool

Use the following script to verify whether a specific file should be referenced by AI assistants:

```bash
node tools/agent-file-checker.js path/to/check
```

If any inconsistencies or ambiguities arise, update this `AGENTS.md` file or `agent_guidelines.md` rather than creating new guidance documents.

---
> Source: [shaiss/circletube](https://github.com/shaiss/circletube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
