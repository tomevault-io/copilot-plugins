## mdpreview

> Guidance for AI coding agents working in this repo or using `mdpreview` as a tool.

# AGENTS.md

Guidance for AI coding agents working in this repo or using `mdpreview` as a tool.
Verified from within **Claude Code**: piped output is escape-free and
width-correct, so an agent can capture and quote it directly.

## Using mdpreview to preview markdown

`mdpreview` renders a Markdown *document* to the terminal — headings become
hierarchy, tables become box-drawn grids, mermaid becomes drawn diagrams — the
way a browser would show it, not the raw source. For an agent this is a better
way to *show a human* what a `.md` file looks like than dumping the source.

When output is piped (which is what an agent captures — no TTY), the default is
already the right mode: **clean monochrome, wrapped to width, zero escape
bytes.** No flags required.

```bash
COLUMNS=100 npx @marttinn/mdpreview file.md   # render; capture and quote the result
COLUMNS=100 npx @marttinn/mdpreview a.md b.md # multiple files, `═══` banner between them
```

What reads well as captured plain text: heading hierarchy, box-drawn tables,
lists, blockquotes, and every mermaid diagram type (flowcharts, sequence
diagrams with loop/alt frames and notes, pie charts, git graphs).

### Flags an agent should and shouldn't use

- `-w N` / `COLUMNS` — render width. Match it to the width you'll show the
  user. Default 80 when piped.
- `--big=blocks` — **safe to add.** h1s render as block-letter banners in plain
  text (no escapes) — a nice title, never noise.
- `--color`, `--images=ascii` — **avoid for text you'll read or quote.** Color
  emits SGR escapes; cell art emits truecolor sextant blocks. Both are noise in
  captured output. Use them only when a human will view the result in a real
  terminal.
- `--tui`, `--watch` — **never from an agent.** They need an interactive TTY and
  will fail or hang in a captured shell. Suggest them to the user instead.

Exit code is non-zero on a missing or unreadable file.

## Working on the codebase

- Runtime is **Node** ≥ 22 — the policy is "every maintained Node line", so
  bump `engines` when a line reaches end of life. The published package is
  plain JS in `dist/` (`npm run build` emits it, `prepack` refreshes it, `bin`
  points there). From a clone, `node src/main.ts file.md` runs straight from
  source on Node ≥ 22.18 via built-in type stripping — so `src/` must stay free
  of enums, namespaces, and parameter properties (`erasableSyntaxOnly` in
  tsconfig enforces it). `npm test` runs the suite (vitest), `npm run typecheck`
  runs `tsc`.
- `src/version.ts` is generated from `package.json` by `tools/sync-version.mjs`
  (wired to the npm `version` hook; a smoke test fails on drift).
- Bun is optional: the same code runs under `bun`, `bun test` works, and
  `bun run build:binary` compiles the standalone `./mdpreview` binary. Only
  use Bun APIs behind a feature check (see `bunTranscoder` in `src/image.ts`).
- Zero runtime dependencies (`"dependencies": {}`) — keep it that way; the
  parser, layout, renderer, diagram engine, pixel font, and PNG decoder are all
  in `src/`.
- The h1 font data (`src/blockfont-data.ts`) is **generated** — edit
  `tools/make-blockfont.ts` and rerun it, never the data file by hand.
- Rendering has golden tests (`test/golden/`). Intended output changes:
  `UPDATE_GOLDEN=1 npm test`, then review the diff.
- Untrusted input is sanitized at parse time (control/escape bytes stripped);
  keep that boundary intact when touching `src/parse.ts`.

---
> Source: [mgranados/mdpreview](https://github.com/mgranados/mdpreview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
