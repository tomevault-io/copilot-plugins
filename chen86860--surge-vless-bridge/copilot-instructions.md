## surge-vless-bridge

> Use Node.js tooling for this project.


Default to using Node.js for this repository.

- Use `node <file>` for compiled JavaScript entry points.
- Use `npm install` to install dependencies.
- Use `npm run <script>` to run package scripts.
- Use `npx <package> <command>` for one-off package binaries.
- Do not introduce Bun-specific APIs or Bun-only workflows.

## Runtime

- Prefer standard Node.js APIs from `node:*`.
- Use `fs/promises`, `path`, `child_process`, and other built-in modules where possible.
- Keep the CLI compatible with regular Node.js execution and npm packaging.

## Build

- Use `tsc` to compile TypeScript into `dist/`.
- The published CLI should run from compiled JavaScript, not directly from Bun.

## Testing

- If tests are added, use Node-compatible tooling and run them through npm scripts.

---
> Source: [chen86860/surge-vless-bridge](https://github.com/chen86860/surge-vless-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
