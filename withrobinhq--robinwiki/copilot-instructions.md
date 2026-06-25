## robinwiki

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Robin**

Robin is an AI-powered second brain that captures thoughts through conversation and structures them into a searchable knowledge base. Users interact with AI (via MCP or web UI), and Robin runs in the background to automatically extract atomic ideas (fragments), classify them into topic clusters (threads), and store everything in a Postgres knowledge base. Every fragment, entry, and wiki is a text row in the database — there is no git-backed markdown store and no filesystem repo.

The server app is the sole application, owning all intelligence, auth, API, MCP, and AI pipeline responsibilities.

The `wiki/` workspace (`@robin/wiki`) is the web frontend — a Next.js 16 application with shadcn/ui. Currently a UI prototype with hardcoded data; will be wired to the core API in a future phase.

**Core Value:** Users can capture raw thoughts and have them automatically structured into searchable, interconnected knowledge — without manual organization.

### Constraints

- **No regressions**: Workspace package boundaries (`@robin/agent`, `@robin/queue`, `@robin/shared`) must be preserved exactly — no flattening
- **Single source**: Migration from existing working code, not a rewrite
- **Workspace layout**: `core/` and `packages/*` are top-level workspace entries, no `apps/` subdirectory
- **Wiki independence**: `wiki/` has its own tsconfig (bundler resolution) and eslint config (eslint-config-next). It does NOT extend tsconfig.base.json or use Biome.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->
## Technology Stack

Technology stack not yet documented. Will populate after codebase mapping or first phase.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [withrobinhq/robinwiki](https://github.com/withrobinhq/robinwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
