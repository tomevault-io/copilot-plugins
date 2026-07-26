## bumpy

> Use Bun instead of Node.js, npm, pnpm, or vite.


Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`

## Testing

Do NOT use `bun test` to run tests. Instead call `bun run test` - which will call the correct test command.

---
> Source: [dmno-dev/bumpy](https://github.com/dmno-dev/bumpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
