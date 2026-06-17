## bun-runtime

> Always use Bun runtime — never npm, npx, or node


# Bun Runtime

This project uses **Bun** as the JavaScript/TypeScript runtime, package manager, and test runner.

## Commands

- `bun install` (not `npm install`)
- `bun add <pkg>` (not `npm install <pkg>`)
- `bun run <script>` (not `npm run <script>`)
- `bunx <pkg>` (not `npx <pkg>`)
- `bun test` (not `jest`, `vitest`, or `npm test`)
- `bun <file.ts>` to run TypeScript directly (not `node`, not `ts-node`)

## Never use

- `npm`, `npx`, `yarn`, `pnpm`
- `node` to execute scripts (use `bun` instead — it runs TypeScript natively)
- `ts-node`, `tsx`

---
> Source: [maziarzamani/spaceduck](https://github.com/maziarzamani/spaceduck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
