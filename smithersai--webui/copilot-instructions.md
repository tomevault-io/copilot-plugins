## webui

> > Update this file when a correction or convention emerges during development.

# smithers-webui — Project Conventions

> Update this file when a correction or convention emerges during development.

## Stack

- **Bun** runtime — `Bun.serve`, `Bun.build`, `Bun.spawn`. Don't reach for Node `http`/`fs` if Bun has it.
- **TypeScript strict** — `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`.
- **incur** for the CLI surface — schemas + MCP + skill discovery for free.
- **Vitest** for tests.

## TypeScript Conventions

- `type` over `interface`.
- No `enum` — use `as const` objects.
- No `readonly` on type properties.
- `exactOptionalPropertyTypes` is on. `foo?: string | undefined`, not `foo?: string`.
- Classes only for errors; everything else is factory functions.
- `const` generic modifier on Zod-schema-accepting functions to preserve literal types.
- camelCase generics (`<const args extends z.ZodObject<any>>`), not `<T>`.
- Short variable names — `options` not `serveOptions`, `fn` not `callbackFunction`.
- No redundant type annotations — let the return type carry it.
- Skip braces for single-statement `if`/`for`.
- Destructure when reading multiple properties.

## Documentation

- README is the contract. Update it before / alongside implementation, not after.
- SKILL.md is for agents. Treat it as a runtime artifact — it ships in the published package.
- JSDoc on every public export.

## Testing

- Vitest. Snapshot tests for deterministic outputs (TOON, MDX → HTML).
- Tests live next to source as `*.test.ts`.
- Type tests in `*.test-d.ts` using `expectTypeOf`.

## Layout

```
src/
├── bin.ts             ← CLI entry, calls Cli.serve()
├── index.ts           ← public API exports
├── render.ts          ← render command implementation
├── server.ts          ← Bun.serve wiring
├── bundle.ts          ← MDX + TSX bundling
├── browser.ts         ← open browser in app mode
└── runtime/
    ├── client.tsx     ← React entry served to the browser
    ├── ui/            ← shadcn-style components (Button, Card, …)
    └── styles.css     ← Tailwind base
```

## Content vs. layout

- All user-visible prose goes in `.mdx`. Never inline content into `.tsx`.
- This is a contract enforced by the skill — code in this repo should follow it too in examples.

## Long-running commands

- `render` is a long-running streaming command. Use `async *run` from incur.
- Yield events (`{ event: 'served', url }`, `{ event: 'opened' }`, `{ event: 'closed' }`) so agents can parse with `--format jsonl`.

## Background-friendliness

- Always log structured events to stdout. The skill spawns us with `> /tmp/log 2>&1 &`, so anything stderr goes the same place.
- Handle SIGINT / SIGTERM cleanly — close the server, let the browser process detach.

## Stdin is the primary input

- The canonical agent pattern is `… | smithers-webui render` (or a heredoc). Agents pay for tool calls; piping keeps render to one Bash call and avoids leaving a temp `.mdx` file behind.
- `--content <path>` exists only for MDX that was already on disk before the agent ran.
- The CLI auto-detects a piped stdin (`!process.stdin.isTTY`) and reads it. If neither stdin nor `--content` is provided, render emits a clear error event and exits.
- When updating SKILL.md or examples, lead with the stdin form. The `--content` form is a fallback; never the headline.

---
> Source: [smithersai/webui](https://github.com/smithersai/webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
