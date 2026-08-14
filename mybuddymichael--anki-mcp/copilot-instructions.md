## anki-mcp

> This is an MCP server for accessing Anki decks, cards, notes, and other data.

# Anki MCP

This is an MCP server for accessing Anki decks, cards, notes, and other data.

## Stack

- Typescript
- Bun
- @modelcontextprotocol/sdk for the Typescript MCP SDK
- yanki-connect (a Typescript library for using AnkiConnect)

## Functionality

The MCP server should allow the LLM to:

- Read Anki decks, notes, and note types
- Create, update, and delete Anki notes

## MCP documentation

- You can find the documentation on MCP generally in docs/model-context-protocol.md
- Docs for the Typescript MCP SDK are in docs/mcp-typescript-sdk.md

## Implementation strategy

- Only implement a single piece of functionality at a time.
  - E.g., JUST reading Anki decks as one piece of functionality.
- Write tests before implementing functionality.
- Write the smallest amount of code necessary to pass the tests.
- We must thoroughly confirm behavior before moving on to the next piece of functionality.

## Typescript

- Use functions instead of classes.
- Prefer small functions and unit tests that don't require mocking.

## Testing

- Add thorough and robust tests before implementing any functionality.
- Run tests to confirm tests fail, then implement the functionality.
- Run tests to confirm tests pass.
- Use `mise run test` to run tests.
- Bun operates in a global space, so proper cleanup is essential.
- Documentation on writing bun tests is in docs/bun-test.md.
- Documentation on mocking modules is in docs/bun-mock.md.
- Rely on unit tests to test small pieces of functionality.
- Write thorough end-to-end / integration tests.
- We can programmatically create an MCP client using the Typescript MCP SDK. See docs/mcp-client-sdk.md.
- There is an actual Anki deck called "Y - MCP Test Deck" with three test cards that can be used for testing.

## Bun

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Bun automatically loads .env, so don't use dotenv.

### APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.md`.

## Misc

- After writing any files, at the end of work but before confirming functionality, format and lint files.
- Use `mise run check`.
- Fix any errors or warnings that arise after running these commands.
- Whenever implementing new functionality, always add appropriate documentation to README.md.
- Finally, run `mise run test` to confirm tests are still passing.

---
> Source: [mybuddymichael/anki-mcp](https://github.com/mybuddymichael/anki-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
