## nostr-agent-interface

> This file gives implementation guidance for contributors working in this repository.

# CLAUDE.md

This file gives implementation guidance for contributors working in this repository.

## Project Overview

This repository is **Nostr Agent Interface**: an extension of the original Nostr MCP Server.

Core principles:

1. Keep one canonical Nostr tool surface (JARC-style tool contracts).
2. Expose that surface through MCP, CLI, and HTTP API.
3. Prefer CLI/API for most operational agent workflows while retaining MCP compatibility.

Current tool count: **48**.

Built with:

1. `snstr`
2. `@modelcontextprotocol/sdk`
3. `zod`

## Common Commands

```bash
# Build TypeScript + refresh tool manifest
bun run build

# Run tests
bun test

# Run interface parity + CLI UX parity
bun run test:parity

# Start MCP mode
bun run start:mcp

# Start CLI mode
bun run start:cli

# Start API mode
bun run start:api
```

## Architecture

### Entry Points

1. `index.ts` - Shared tool registration and MCP host for the common tool surface.
2. `app/index.ts` - Interface launcher (`mcp`, `cli`, `api`).
3. `app/cli.ts` - CLI wrapper over shared tool contracts.
4. `app/api.ts` - HTTP wrapper over shared tool contracts.
5. `app/mcp-client.ts` - Wrapper/client bridge used by CLI/API to invoke the shared tool surface.

### Tool Modules

Feature modules expose both:

1. Business logic.
2. Schema contracts reused across transports.

Primary directories:

1. `profile/`
2. `note/`
3. `event/`
4. `social/`
5. `relay/`
6. `dm/`
7. `zap/`
8. `utils/`

### Cross-Cutting Patterns

1. Accept both hex and NIP-19 formats where applicable (`npub`, `nsec`, etc.).
2. Normalize keys through shared helpers before signing or querying.
3. Keep output semantics transport-consistent (MCP/CLI/API parity).
4. Close relay resources in `finally` paths.

## Testing

Tests run on Bun and include:

1. Unit tests per module.
2. Integration tests with in-memory relay (`utils/ephemeral-relay.ts`).
3. Parity tests across MCP/CLI/API.

Primary docs:

1. `docs/testing.md`
2. `__tests__/README.md`

## Artifact Contract

Tool schema artifact:

1. `artifacts/tools.json`

Generation:

```bash
bun run generate:tools-manifest
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AustinKelsay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
