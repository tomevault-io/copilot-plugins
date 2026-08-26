## madonna-quotation

> Quotation builder for Madonna Home Solutions (legal entity: Madonna Industries),

# Madonna Quotation App

Quotation builder for Madonna Home Solutions (legal entity: Madonna Industries),
Patiala. Two panes: catalogue cards on the left, a live A4 PDF preview on the
right. Full plan: `docs/PLAN.md`.

## This is NOT the Next.js you know

Next 16.x. APIs, conventions and file structure differ from training data. Read
the relevant guide in `node_modules/next/dist/docs/` before writing app code.

Already established from those docs:

- **Turbopack is the default** for `next dev` and `next build`. No `--turbopack`
  flag. A stray `webpack` config in `next.config.ts` makes the build *fail*.
- **`middleware.ts` is now `proxy.ts`**, exporting `proxy()` rather than
  `middleware()`. Runtime is `nodejs` and is not configurable. Config flags
  renamed to match (`skipMiddlewareUrlNormalize` → `skipProxyUrlNormalize`).
  This is how the shared-password gate gets built.
- Request-time APIs (`params`, `searchParams`, `cookies`, `headers`) are
  **async** — await them.
- ESLint uses flat config; `next lint` is removed, run `eslint` directly.

## Non-negotiables

- **The document is a real PDF, never HTML.** One component
  (`src/document/`) renders in the browser for the live preview and on the
  server for download. There is no second implementation, ever — a divergent
  HTML twin is the specific failure this design exists to prevent.
- **Quote lines carry frozen copies** of every price and printed string, taken
  when the card was added. Editing a catalogue default must never alter a quote
  that already exists. This is the central guarantee of the whole app.
- **Money is integer paise.** No floats anywhere.
- **Indian digit grouping** (`1,42,000`) via `Intl.NumberFormat('en-IN')`.
- **One date format** everywhere: `30 July 2026`.
- A product and its description must **never split across a page break**.
- The browser never talks to Supabase directly — no Supabase Auth means RLS has
  no identity to enforce against, so all data access goes through server code.

## Database

Supabase, but **not used as a Supabase app** — its Data API is off, so
`supabase-js` cannot work and there is no public endpoint. Connect over plain
Postgres through the transaction pooler, from `src/lib/db.ts` only. That file
imports `server-only`, which turns "the browser reached the database" from a
review comment into a build error.

- `npm run db:check` — proves the connection, and that it can create tables.
- `.env.local` holds `DATABASE_URL` locally; Vercel holds its own copy.
- Prepared statements are disabled: the transaction pooler hands a different
  backend to each statement, so a plan from one is meaningless on the next.

## Traps already paid for

Each of these cost real time to find. They fail quietly, not loudly.

- **A `flex` value that was right in a row is destructive in a column.**
  `flex: 1` on a child of a row sets its width; on a child of a column it sets
  its HEIGHT. This happened twice in one restructure: once it clipped every
  spec line after the first, once it drew a block heading on top of its own
  first line. Both rendered without error and passed every text-based check.
- **`page.render({ canvas, canvasContext })` in pdf.js hangs.** The canvas must
  be null if the context is supplied. Pass `canvas` alone.
- **pdf.js drives rendering on `requestAnimationFrame`,** which a hidden tab
  never fires, so a render promise can stay pending forever. The preview
  re-runs on `visibilitychange` for that reason.
- **Tailwind silently drops arbitrary values it cannot compile** — a nested
  `calc()` inside `min()` produced no class at all and the pane split in half.
- **`aspect-ratio` cannot size a flex item** whose content defines its size;
  the resolution is circular. Use an explicit width.
- **A `fixed` element only repeats from the page its flow position falls on.**
  Declared last, the footer went missing from page one. Declare it early.
- **On a stretched page nothing may be absolutely positioned** — the page's
  height comes from its content, and absolute elements sit outside that
  measurement, so the footer lands on the last line.

## Verifying

`npm run verify` renders 19 samples and checks them. Two lessons are baked in:

- **Check geometry, not just text.** Overlapping runs extract perfectly, so a
  page can be unreadable while every string assertion passes. There is a check
  that compares baselines and horizontal extents for exactly this.
- **Prove a new check can fail.** Reintroduce the bug, watch it go red, then
  restore. Two checks written here passed vacuously until that was done.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [tejsav/madonna-quotation](https://github.com/tejsav/madonna-quotation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
