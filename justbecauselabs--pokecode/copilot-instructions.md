## pokecode

> Default to using Bun instead of Node.js.

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
- Bun.$`ls` instead of execa.

### File System Operations

- Use `Bun.file(path)` instead of `fs.readFile()` or `fs.writeFile()`
- Use `Bun.Glob` for pattern matching instead of `fs.readdir()` with filtering
- Use `file.exists()` instead of `fs.stat()` for existence checks
- Use `file.text()` for reading text content
- Use `file.arrayBuffer()` for binary content
- Example: `const content = await Bun.file('./file.txt').text()`

## Typescript

### Best Practices

- Always use the most strict tsconfig settings
- **NEVER use `any`, `unknown`, or `as any` - EVER. This is forbidden.**
- **NEVER use type assertions with `as` - types should be inferred or properly typed**
- **If you find yourself needing `as any` or type assertions, the types are wrong and need to be fixed properly**
- Use params: { name: type } for functions, ie `function add(params: { a: number, b: number }): number` over `function add(a: number, b: number): number` as it improves readability
- Prefer type narrowing with type guards over type assertions
- Use proper Zod schemas and inferred types instead of casting
- If TypeScript is complaining, fix the types - don't silence it with assertions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justbecauselabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
