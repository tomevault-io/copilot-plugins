## agent-worker

> TypeScript CLI built with Bun. Polls Linear for tickets and dispatches them to an agent harness.

# Agent Worker

TypeScript CLI built with Bun. Polls Linear for tickets and dispatches them to an agent harness.

## Stack

- Runtime: Bun
- Language: TypeScript
- Testing: `bun test`
- Dependencies: @linear/sdk, zod, yaml

## Workflow

When solving a ticket:

1. Write the code to solve the ticket
2. Run `bun test` and fix any failures
3. Review your changes for bugs, security issues, and code quality — use CodeRabbit if available
4. Fix any issues found in the review
5. Run `bun test` again to confirm fixes didn't break anything

## Conventions

- No classes — use plain functions and interfaces
- Validate config with zod schemas (`src/config.ts`)
- Executors implement the `CodeExecutor` interface (`src/pipeline/executor.ts`)
- Providers implement the `TicketProvider` interface (`src/providers/types.ts`)
- Hooks are shell commands run via `src/pipeline/hook-runner.ts`
- Tests live in `test/` mirroring `src/` structure

---
> Source: [owainlewis/agent-worker](https://github.com/owainlewis/agent-worker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
