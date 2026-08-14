## ags-api-mcp-server

> > **Note:** This is a **project-specific** CLAUDE.md. It applies only to this project.

# CLAUDE.md — AGS API MCP Server (Authoritative)

> **Note:** This is a **project-specific** CLAUDE.md. It applies only to this project.
> For personal preferences across all projects, use `~/.claude/CLAUDE.md`.

---

## Authority & Usage Rules

**This document is the AUTHORITATIVE source for understanding this project.**

Treat this file as the **single source of truth**. Do NOT rediscover information that is already documented here.

**Required behavior:**
* **Trust this file over repository inference** — If something is documented here, accept it as fact
* **Do NOT rescan the repository** to rediscover high-level context already provided
* **Only read additional files when explicitly required for the task** — Don't explore "just in case"
* **If something is not described here, assume it is out of scope** unless the task directly requires it

---

## Project Summary

**Purpose:** MCP server that provides AI assistants (VS Code Copilot, Cursor, Claude) with access to AccelByte Gaming Services APIs through OpenAPI integration.

**Stack:** Node.js, TypeScript, Express 5, MCP SDK (@modelcontextprotocol/sdk), Zod, Docker

---

## Architecture Overview

```
/src
  /v2              → V2 implementation (current, production-ready)
    /auth          → Authentication middleware & routes (Bearer token)
    /mcp           → MCP server, tools, prompts, elicitations
      /tools       → MCP tool implementations
      /prompts     → MCP prompt implementations
    index.ts       → V2 entry point
    express.ts     → Express app setup
    config.ts      → Configuration
  /tools           → OpenAPI tools (shared)
  /prompts         → Workflow prompts (shared)
  *.ts             → V1 implementation (legacy, stdio + http)
/openapi-specs     → AccelByte OpenAPI specifications
/docs              → Documentation
/tests             → Test files
/tools/oauth       → OAuth helper tool (separate Docker service)
```

**V2 is the primary implementation.** V1 exists for backward compatibility (stdio transport, server-managed OAuth).

**Data flow (V2):** HTTP Request → Express → Auth Middleware (Bearer token) → MCP Server → Tool Handler → AccelByte API

---

## Invariants, Conventions & Preferences

### Technical Invariants

* **V2 is stateless** — No server-side sessions or token storage; client manages tokens
* **V2 is HTTP-only** — No stdio transport (V1 supports both)
* **Authentication:** Bearer token in Authorization header (V2), server-managed OAuth (V1)
* **Validation:** Zod schemas throughout V2
* **OpenAPI specs:** Located in `/openapi-specs/`, processed at build time

### Code Conventions

* **Linting:** ESLint configured (`.eslintrc.cjs`)
* **Formatting:** Prettier configured
* **Type Safety:** TypeScript strict mode
* **Module System:** ES Modules (`"type": "module"`)
* **Testing:** Node.js built-in test runner with tsx (`pnpm test:unit`)

### Project Preferences

* **V2 code lives in `/src/v2/`** — Do not mix with V1 code in `/src/*.ts`
* **Commits:** Conventional commits format (`feat:`, `fix:`, `refactor:`, etc.)

---

## Boundaries & Constraints

### Off-Limits (Do Not Modify)

Unless explicitly requested:

* `/node_modules/` — Package dependencies
* `/dist/` — Build outputs (regenerate with `pnpm build`)
* `/.git/` — Git metadata
* `/openapi-specs/` — Source OpenAPI specs (processed, not edited)
* `/docs/v1/README.md` — V1 legacy archive (single file)

### Behavioral Constraints

Unless explicitly requested, avoid:

* Full repository scans or architecture analysis
* Suggesting stack replacements or major refactors
* Modifying V1 code when working on V2 features
* Touching files outside the current task area

### Locked-In Assumptions

* V2 stateless architecture is intentional (see `docs/ARCHITECTURE.md`)
* Express 5 is the chosen framework
* MCP SDK patterns are followed per specification

---

## Documentation Map

* `CLAUDE.md` ← **Authoritative architectural overview** (you are here)
* `README.md` → Main project documentation, hosted-server install
* `INSTALL.md` → AI-assistant install guide for the hosted server
* `docs/ARCHITECTURE.md` → Architecture, security (JWT, SSRF, rate limits), render tools, AFS operations
* `docs/ENVIRONMENT_VARIABLES.md` → Environment configuration
* `docs/DEVELOPMENT.md` → Self-hosting, development, testing, and Docker deployment
* `docs/v1/README.md` → V1 legacy archive (single file)
* `CHANGELOG.md` → Version history

---

## Working Approach

When completing tasks:

1. **Start here** — Use CLAUDE.md as the authoritative context foundation (do NOT rescan the repository for information already documented)
2. **Read targeted files** — Only open files directly relevant to the task
3. **Minimal changes** — Prefer focused edits over broad refactors
4. **Follow patterns** — Match existing code style and architecture
5. **Ask only when blocked** — Proceed with constraints above if information seems missing but isn't required

**If the task truly cannot be completed without more info, ask specific questions.**

---

## Common Commands

```bash
pnpm build          # Compile TypeScript
pnpm start          # Run V2 server
pnpm dev            # Watch mode
pnpm test:unit      # Run unit tests
pnpm lint           # Lint V2 code
pnpm format         # Format code
pnpm inspect        # Run with MCP Inspector
```

---

---
> Source: [AccelByte/ags-api-mcp-server](https://github.com/AccelByte/ags-api-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
