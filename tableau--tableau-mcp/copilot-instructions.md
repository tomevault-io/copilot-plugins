## tableau-mcp

> Tableau MCP project context, architecture overview, and core principles


# Tableau MCP — Project Context

## What This Is
Official Tableau MCP server (`tableau/tableau-mcp`). Provides MCP tools
for AI applications integrating with Tableau Cloud and Server via
REST API, VDS API, Metadata API, and Pulse API.

## Role Context
Auth lead on this project. All auth-related code requires architectural
rigor — not just working code.

## Core Principles
- Deterministic control over prompt hope for all auth paths
- Programmatic enforcement for security-critical operations
- Structured errors always — never silent failures
- Understand before building — plan before coding

## Architecture Overview

### Source Layout
```
src/
├── config.ts              # Config loading from env
├── server.ts              # MCP Server class
├── restApiInstance.ts      # REST API lifecycle, auth, useRestApi()
├── index.ts               # Entry point (stdio/http)
├── tools/                 # MCP tools (factory pattern)
│   ├── tool.ts            # Base Tool class
│   ├── tools.ts           # Tool factory registry
│   ├── toolContext.ts     # TableauRequestHandlerExtra type
│   └── toolName.ts        # Tool name constants + groups
├── server/                # HTTP server & auth middleware
│   ├── express.ts         # Express setup, middleware chain
│   ├── passthroughAuthMiddleware.ts
│   └── oauth/             # OAuth provider, tokens, scopes
├── sdks/tableau/          # Tableau REST API client (Zodios)
├── utils/                 # Shared utilities
├── telemetry/             # Telemetry
└── logging/               # Structured logging
```

### Key Libraries
- `ts-results-es` — `Result<T, E>`, `Ok()`, `Err()` for tool callbacks
- `zod` — Schema validation for params, auth tokens, configs
- `@zodios/core` — Type-safe REST API client for Tableau APIs
- `jose` — JWT/JWE operations for OAuth tokens
- `vitest` — Test framework (NOT jest)

### Tool Registry (All 16 Tools)
Datasource: `list-datasources`, `get-datasource-metadata`, `query-datasource`
Workbook: `list-workbooks`, `get-workbook`
View: `list-views`, `get-view-data`, `get-view-image`
Pulse: `list-all-pulse-metric-definitions`,
  `list-pulse-metric-definitions-from-definition-ids`,
  `list-pulse-metrics-from-metric-definition-id`,
  `list-pulse-metrics-from-metric-ids`,
  `list-pulse-metric-subscriptions`,
  `generate-pulse-metric-value-insight-bundle`,
  `generate-pulse-insight-brief`
Content Exploration: `search-content`

Auth touches ALL of these. Changes to auth affect every tool downstream.

## Error Handling Patterns
- `getExceptionMessage(error: unknown)` — safe error text extraction
- `getHttpStatus(error: Error)` — HTTP status from AxiosError or PulseDisabledError
- `invariant(condition, message)` — internal precondition assertions
- `src/utils/retry.ts` — exponential backoff (3 retries, jitter)
- Zod validation errors → non-error results with warning + raw data
  (so users don't get stuck on schema strictness)

## PR Standards
PRs must include:
- What problem this solves
- What alternatives were considered
- Auth impact assessment (if auth-adjacent)
- Test coverage for new paths

## What Good Looks Like
- Auth errors surface cleanly with enough context for the calling agent
  to recover or escalate
- No tool can execute data operations without verified auth
- A new contributor can understand the auth flow from the code alone

---
> Source: [tableau/tableau-mcp](https://github.com/tableau/tableau-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
