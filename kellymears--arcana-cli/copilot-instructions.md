## arcana-cli

> Arcana CLI — a terminal tarot reading application built with Ink (React for CLIs). TypeScript, ES Modules, Node 24+.

# CLAUDE.md

## Project overview

Arcana CLI — a terminal tarot reading application built with Ink (React for CLIs). TypeScript, ES Modules, Node 24+.

## Architecture

- `src/cli.tsx` — Entry point. Calls Ink's `render()` with the root `<App />` component.
- `src/app.tsx` — Root component. All UI flows from here.
- `dist/` — Compiled output (gitignored). The `dist/cli.js` binary is the executable.

Ink components use `<Box>` for layout (flexbox) and `<Text>` for styled text output. Think React but rendering to the terminal.

## Key conventions

- **ESM only** — `"type": "module"` in package.json. All imports use `.js` extensions (NodeNext resolution requires this even for `.tsx` source files).
- **Automatic JSX runtime** — `jsx: "react-jsx"` in tsconfig. Do not add `import React` to component files.
- **Strict TypeScript** — strict mode is enabled.
- **Import sorting** — eslint-plugin-perfectionist enforces natural sort order on imports, exports, object keys, and JSX props. Sort alphabetically.
- **Prettier** — double quotes, semicolons, trailing commas, 2-space indent, 80 char width.

## Commands

- `npm run build` — Compile TypeScript to `dist/`
- `npm run dev` — Watch mode
- `npm start` — Run the CLI
- `npm run lint` — ESLint
- `npm run format` — Prettier write
- `npm run format:check` — Prettier check

## Verification

After making changes, always run:

```sh
npm run build && npm start && npm run lint && npm run format:check
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kellymears) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
