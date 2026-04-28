## closedloop-electron

> - `apps/desktop/` -- Electron desktop app (localhost HTTP gateway, cloud relay, tray UI)

# ClosedLoop Electron Monorepo

## Structure

- `apps/desktop/` -- Electron desktop app (localhost HTTP gateway, cloud relay, tray UI)

## Useful Commands

Run `just` to see all available recipes. Key ones:

- `just desktop-dev` -- build and start Electron
- `just desktop-no-auth` -- start with gateway auth disabled (dev only)
- `just desktop-debug-auth` -- start with debug token minting enabled
- `just desktop-lint` -- run ESLint
- `just desktop-typecheck` -- run TypeScript type checking
- `just desktop-test` -- run tests

## Breaking Changes

Any breaking change to APIs, contracts, or interfaces (HTTP gateway routes, cloud relay messages, IPC bridge, persisted store schemas, etc.) requires both of the following before merging:

1. **Legacy migration logic** so existing users are not broken on upgrade. Detect the old shape and translate it to the new shape at the boundary; do not assume users have already migrated.
2. **A ClosedLoop ticket** created via the ClosedLoop MCP (`mcp__closedloop__create-feature`) to track removing the legacy migration code at a later date. Reference the ticket ID in a comment next to the migration logic so it can be found and deleted when the ticket is worked.

## Commit Messages

Follow the format in `.gitmessage`. The subject line must be `<TICKET>: <description>` where TICKET is extracted from the branch name (e.g. `FEAT-68: add no-auth dev mode`). Include bullet-point body, Testing, and Risks sections.

---
> Source: [closedloop-ai/closedloop-electron](https://github.com/closedloop-ai/closedloop-electron) — distributed by [TomeVault](https://tomevault.io/claim/closedloop-ai).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
