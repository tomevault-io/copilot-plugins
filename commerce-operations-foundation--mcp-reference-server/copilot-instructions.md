## mcp-reference-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Commerce Operations Foundation MCP Server - implements the Order Network eXchange Standard (onX) providing a standardized interface between AI agents and fulfillment systems via the Model Context Protocol (MCP).

## Project Structure

```
/server          - Core MCP server implementation (main development here)
/adapter-template - Template for creating custom fulfillment adapters
/schemas         - JSON Schema definitions for domain models
/backends        - Backend-specific capability manifests
/docs            - Specification and integration guides
```

## Development Commands

All primary development happens in `/server`. Run these from the `server/` directory:

```bash
# Build
npm run build          # Compile TypeScript
npm run dev            # Run with hot reload (vite-node)
npm start              # Run production build

# Test
npm test               # Build + run all tests
npm run test:unit      # Unit tests only
npm run test:integration  # Integration tests
npx vitest tests/unit/adapters/adapter-factory.test.ts  # Single test file
npx vitest tests/unit --grep "pattern"                  # Tests matching pattern

# Quality
npm run lint           # ESLint
npm run format         # Prettier
npm run typecheck      # Type checking
```

For adapter development (`adapter-template/`):
```bash
npm run build          # tsc
npm run dev            # tsc --watch
npm test               # Jest with ES modules
```

## Architecture

### MCP Server (server/)

Three-layer architecture:
1. **Protocol Layer** - MCP SDK integration, stdio transport, request routing
2. **Service Layer** - `ServiceOrchestrator` facade, `AdapterManager`, health monitoring, validation
3. **Adapter Layer** - `AdapterFactory` with dynamic loading (built-in, NPM, or local adapters)

**Tools**: 5 action tools + 7 query tools defined in `src/tools/`, registered via `registerTools()` in `ToolRegistry`

**Configuration**: Environment variables override config file which overrides defaults. Key vars: `ADAPTER_TYPE`, `ADAPTER_NAME/PACKAGE/PATH`, `LOG_LEVEL`

See [server/CLAUDE.md](server/CLAUDE.md) for detailed server architecture.

### ES Module Requirements

- All imports use `.js` extension
- `type: "module"` in package.json
- Use `import`/`export`, not `require`

---

# MCP Server Principles

The MCP server acts as an adapter between a generic tool schema and a concrete backend. It does not implement business logic, does not infer missing data, and remains fully backend-agnostic.

---

## Core Principles

### 1. Backend-Agnostic Behavior

- The MCP server must not assume or enforce backend-specific requirements.
- It implements only the tool specification and passes requests/responses through.
- It performs no additional validation beyond the minimal schema validation defined by the MCP tool.

### 2. No Business Logic Inside the MCP Server

- The MCP server must not fill in missing fields.
- It must not generate placeholder values (e.g., "Unknown", "N/A").
- It must not attempt to "fix" or "repair" incomplete requests.
- It must not apply heuristics to guess user intent or missing data.

### 3. Transparent Error Propagation

- Backend errors must be returned to the agent exactly as received.
- The MCP server must not modify, hide, or reinterpret backend errors.
- The agent is responsible for asking the user for missing information.

---

## Error Handling Philosophy

### Behavior When Required Fields Are Missing

If the backend requires additional fields (e.g., `customerName`) that are not mandatory in the MCP schema:

1. The MCP server forwards the request unchanged to the backend.
2. The backend returns an error.
3. The MCP server returns this error unchanged to the agent.
4. The agent asks the user for the missing information.

### General Error Handling Rules

- The MCP server must propagate backend errors transparently.
- It must never attempt to repair or complete invalid requests.
- Backend-specific error codes and messages should pass through unchanged.

---

## Capability Definition

Each backend integration must define its own **capability manifest** that specifies:

1. **Actions** (write operations) - Methods that modify state
2. **Queries** (read operations) - Methods that retrieve data without side effects
3. **Error documentation** - Backend-specific error handling rules

See the `backends/` directory for backend-specific capability definitions.

---

## Usage Rules for Claude

These rules apply to all MCP server integrations:

- Claude MUST use only the methods defined in the backend's capability manifest.
- Claude MUST NOT invent new methods or assume additional capabilities.
- Claude MUST choose the correct method based on user intent (read vs. write).
- Claude MUST follow all error-handling rules defined in the backend's error documentation.
- Claude MUST NOT attempt to bypass the MCP server or call backend endpoints directly.

---

## Backend Integrations

| Backend | Capability Manifest |
|---------|---------------------|
| [your-backend] | [backends/your-backend/CAPABILITIES.md](backends/your-backend/CAPABILITIES.md) |

> To add a new backend, create a new directory under `backends/` with a `CAPABILITIES.md` file following the same structure.

---
> Source: [commerce-operations-foundation/mcp-reference-server](https://github.com/commerce-operations-foundation/mcp-reference-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
