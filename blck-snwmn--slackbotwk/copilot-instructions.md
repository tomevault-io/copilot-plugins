## slackbotwk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Slack Bot application running on Cloudflare Workers. Provides a reception/inquiry system with interactive modals and slash commands for questions, feature requests, bug reports, and general inquiries.

## Tech Stack

- Runtime: Cloudflare Workers
- Language: TypeScript (strict mode)
- Slack SDK: @slack/bolt
- Package Manager: Bun
- Testing: Vitest (unit tests co-located with source files)
- Linting/Formatting: oxlint, oxfmt

## Commands

```bash
bun run dev          # Local development server (wrangler dev)
bun run deploy       # Deploy to Cloudflare Workers
bun run test         # Run tests with Vitest
bun run lint         # Lint with oxlint
bun run lint:fix     # Auto-fix lint issues
bun run fmt          # Format with oxfmt
bun run fmt:check    # Check formatting
bun run cf-typegen   # Generate Cloudflare types
```

## Architecture

```
src/
├── index.ts          # Worker entry point (fetch handler)
├── app.ts            # App initialization, handler registration
├── receiver.ts       # Custom Receiver for Cloudflare Workers compatibility
├── types/env.ts      # Environment variables interface
├── handlers/
│   ├── commands.ts   # Slash commands (/hello, /help, /reception)
│   ├── events.ts     # Event handlers (app_mention)
│   └── actions.ts    # Interactive component handlers (buttons, modal submissions)
├── blocks/
│   ├── messages.ts   # Message block definitions (help, reception menu)
│   └── modals.ts     # Modal view definitions (question, request, bug, other)
└── validators/
    └── datetime.ts   # Date/time validation for bug reports
```

**Request Flow:**
1. `index.ts` receives incoming requests
2. `receiver.ts` validates signature, parses body, handles URL verification
3. `app.ts` routes to appropriate handler based on payload type
4. Handlers in `handlers/` process and respond

## Key Patterns

- **Modal Metadata**: Channel IDs passed via `private_metadata` JSON in modal views
- **Validation**: Date/time validation prevents future dates in bug reports (Japanese error messages)
- **Rich Text**: Bug reports and feature requests support rich text blocks
- **Early Acknowledgment**: `processBeforeResponse: true` for timely Slack responses

## Environment Variables

Required secrets (set via `wrangler secret put`):
- `SLACK_SIGNING_SECRET` - From Slack App Credentials
- `SLACK_BOT_TOKEN` - From OAuth & Permissions (requires: commands, app_mentions:read, chat:write)

## Adding New Features

**New Slash Command:**
1. Add handler in `src/handlers/commands.ts`
2. Register command in Slack App configuration

**New Modal:**
1. Define modal view in `src/blocks/modals.ts`
2. Add action handler for submission in `src/handlers/actions.ts`

**New Event:**
1. Add handler in `src/handlers/events.ts`
2. Subscribe to event in Slack App configuration

## Testing

Unit tests are co-located with source files (`*.test.ts`).

**Test Structure:**
- `src/receiver.test.ts` - Signature verification, request handling
- `src/blocks/*.test.ts` - Message/modal block definitions
- `src/validators/*.test.ts` - Validation logic

## Known Issues

### vitest-pool-workers and @slack/bolt Compatibility

`@slack/bolt` internally uses `node:os`, which causes `@cloudflare/vitest-pool-workers` tests to fail.

**Related GitHub Issues:**
- [#9719](https://github.com/cloudflare/workers-sdk/issues/9719) - node:os module resolution issue
- [#7324](https://github.com/cloudflare/workers-sdk/issues/7324) - Node module compatibility issue

**Current Workaround:**
- Use standard Vitest instead of vitest-pool-workers
- Test only parts that don't depend on `@slack/bolt` (blocks/, validators/, parts of receiver)

**Note:** Check the above issues for updates before modifying tests. This may be fixed in the future.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blck-snwmn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
