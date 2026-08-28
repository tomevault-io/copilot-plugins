## ait-tagforge

> Context an agent working on this repo won't get from the code alone.

# Agent notes

Context an agent working on this repo won't get from the code alone.
Full strategic/design context lives in [`PRODUCT.md`](./PRODUCT.md) —
read it first for register, users, positioning, brand personality, and
design principles.

## Toolchain: Bun only

Never use `node`, `npm`, or `npx`, anywhere, including one-off scripts.
Always `bun --bun <script>` / `bunx --bun <tool>`. Bun sometimes copies
itself to a temp path named `node.exe` to satisfy tools that hard-invoke
`node` on Windows — that's still Bun, not a separate runtime; don't
"fix" it.

| Task | Command |
| --- | --- |
| Install dependencies | `bun install` |
| Add a package | `bun --bun add <pkg>` |
| Run a CLI tool | `bunx --bun <tool>` |
| Dev (backend) | `bun --bun server.ts` |
| Dev (frontend) | `bun --bun dev` |
| Build | `bun --bun run build` |
| Run one file | `bun --bun <file.ts>` |
| Run tests | `bun test` |

Dev runs as **two processes**: `server.ts` (Bun.serve backend, port
3000) and Vite (frontend, port 5173, proxies `/api` to the backend).
Production is one process: `bun --bun run build && bun --bun server.ts`.

## Data modes

Backend defaults to **mock mode** (fixture attendees in
`src/mock/attendees.ts`, no network). Setting `AIT_AGENT_API_KEY` (see
[`.env.example`](./.env.example)) switches to **live mode**, calling
the real AI Tinkerers Agent API. The Agent API key must never reach
the browser — `server.ts` is the only thing that holds it.

There is **no structured company/employer field** anywhere in that
API (checked `rsvps/get`, `clients/get`, `client_profile_search`'s
`profile_answers`). `companyGuess` in `src/lib/attendee.ts` is a
best-effort parse of the title, falling back to a bio-text scan — it
is always meant to be user-editable, not authoritative.

## Name tag layout

`src/lib/typst/tag.typ` is a real, standalone Typst file, not a
string built up in TypeScript. `render.ts` compiles it with attendee
data passed through Typst's own `sys.inputs` mechanism (`inputs: {
data: JSON.stringify(...) }`), the same mechanism `typst compile
--input` uses, so the file is independently editable/testable with
the real Typst CLI. `src/lib/typst/tag-data.ts` only shapes attendee
data into that JSON payload; it holds no layout logic. See the
comment at the top of `tag.typ` for the local `typst watch` workflow.

`src/lib/typst/sample-data.json` (gitignored, copy it from the
committed `sample-data.example.json`) is a personal scratch fixture
`tag.typ` falls back to when compiled standalone with no `--input`.
The running app never reads it, don't wire it into any app code path.

Fonts (`public/fonts/*.ttf`, committed) must be loaded explicitly:
Typst's rendering engine has its own font database, unrelated to the
browser/CSS font stack, so a `<link>`/`@font-face` does nothing for
it. `render.ts` wires them in via `loadFonts()` from
`@myriaddreamin/typst.ts`; the real family/weight each file resolves
to is whatever its own name table says, not its filename, so verify
with `typst fonts --variants --font-path public/fonts` (not by
assumption) before changing `font:`/`weight:` values in `tag.typ`.

`buildTagData()` splits role tags into two arrays the template styles
differently: `presenting-labels` (Speaker, Science Fair) render as a
filled pill; everything else in `tag-labels` (organizer, sponsor, ...)
renders outlined. Keep that split when adding new role tags.

## Printer support

Not locked to the Munbyn MC240. `src/lib/printer/known-services.ts`
is an open registry of known BLE UART service UUIDs shared across
most cheap thermal/label printers. Adding a new printer means adding
a UUID to that registry, not writing a new driver. TSPL and ESC/POS
encoders are hand-written (`src/lib/printer/tspl.ts`,
`src/lib/printer/escpos.ts`) rather than depending on the one
available npm driver, which lists 0/95 devices verified.

Web Bluetooth only works in Chrome/Edge, desktop or Android — never
iOS, in any browser (Apple requires WebKit there, which never
implemented it). `src/screens/PrinterLab.tsx` is the tool for probing
a new printer against this before trusting the main flow.

[`PRINTERS.md`](./PRINTERS.md) is the living compatibility doc
(confirmed-working table, how to test a new printer, known
limitations) — update it, not just the registry, whenever a printer
gets tested against real hardware.

## Tests

`bun test` (Bun's built-in runner, no extra dependency) runs every
`*.test.ts` file, colocated next to the module it covers. Coverage is
the pure/deterministic logic: attendee parsing (`src/lib/attendee.ts`),
the live-API mapper (`src/lib/live-attendee.ts`), the tag data shaper
(`src/lib/typst/tag-data.ts`), and the printer byte encoders
(`src/lib/printer/*.ts`). Screens and anything touching real Web
Bluetooth/camera/WASM, or the actual Typst layout in `tag.typ` itself,
stay covered by [manual + `verify-ui` skill](#verifying-ui-changes)
checks instead, not `bun test`.

`server.ts` calls `Bun.serve` at import time, so its route handlers
aren't imported directly in tests. The one non-trivial piece of logic
it had, mapping a live API response onto this app's `Attendee` shape,
lives in `src/lib/live-attendee.ts` instead, specifically so it can be
unit-tested without booting a server.

## Git commits

Conventional Commits format. Never add a Claude/AI co-author trailer.
Never put PII (real names, companies, emails from live data) in a
commit message — use a placeholder. Always confirm the message with
the user before committing.

## Verifying UI changes

Playwright's own `launch()`/`connectOverCDP()` hang indefinitely under
Bun on this machine (a Bun-on-Windows `node.exe` shim issue). Don't
use Playwright here. Use the `verify-ui` skill instead — it drives a
real Chromium instance over raw CDP JSON-RPC via a plain `WebSocket`
from a Bun script. Never clean up with a blanket `taskkill /IM
<image>`; kill only the specific PID(s) spawned for the check.

## Documentation cross-references

Always use real markdown links (`[text](path.md#anchor)`) between
docs in this repo, never plain-text file/section mentions. Applies to
`README.md`, `PRINTERS.md`, `AGENTS.md`, `CLAUDE.md`, `PRODUCT.md`
alike.

---
> Source: [cr2007/ait-tagforge](https://github.com/cr2007/ait-tagforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
