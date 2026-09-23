## puzzle-devtools

> Read this before changing anything here. `README.md` explains the product; this file

# Puzzle DevTools — agent knowledge base

Read this before changing anything here. `README.md` explains the product; this file
records the constraints and the traps.

## What this repo is

A Chrome MV3 DevTools extension that inspects [Puzzle](https://github.com/magic-spells/puzzle)
apps. The framework ships a dev-only **runtime bridge** (`client-runtime/devtools.js`,
decision D100); this repo ships everything else. The two halves are joined ONLY by the
wire protocol — never by shared code.

**The panel UI is itself a Puzzle app** (`panel/`), compiled by the real compiler. That
is deliberate dogfooding: protocol messages upsert into the panel's own Puzzle store and
the panels are ordinary reactive views. When something is awkward to build here, that is
a finding about the framework, not a reason to reach outside it.

## Commands

```bash
npm install            # @magic-spells/puzzle from npm; provides node_modules/.bin/puzzle
npm run build          # panel + dist-extension/
npx vitest run         # the full suite — must be green before any commit
npm run serve:fixture  # the synthetic bridge at :5177
```

To develop against an unpublished framework checkout:
`cd <puzzle>/compiler && go build -o ../puzzle ./cmd/puzzle`, then
`PUZZLE_BIN=<puzzle>/puzzle npm run build`.

## Invariants that are easy to break

- **The protocol is the only interface.** `constellation/doc/DOC-SPEC.md` §55 in the
  framework repo is the authority; `protocol/constants.js` transcribes it and
  `tests/protocol.test.js` pins the names literally. Never reach into framework
  internals from this repo, and never assume a version — `hello` negotiates.
- **The message set grows ADDITIVELY; do not bump `PROTOCOL_VERSION` to add one.**
  Unknown events fall through `receive()`'s default case into the ring and unknown
  requests fail per-call with `{ error }`, so both ends already tolerate names they do
  not know. A bump puts every already-published app into the hard `MISMATCH` state and
  blanks *every* panel — a far worse regression than one panel reporting that this
  runtime has no profiler. `tests/protocol.test.js` pins v1 and the additive rule.
- **Only the Performance panel polls, and only while recording.** SPEC §55 defines no
  per-render event on purpose: the page hook buffers 500 messages pre-attach and the
  panel ring holds 200, so a render firehose would overrun both and evict the events the
  other panels live on. Render counts are therefore PULLED (`snapshot:profile`, once a
  second while recording, zero requests otherwise) and only `perf-warning` is pushed. Do
  not add a second polling path; use the counter-and-debounce mechanism instead.
- **`resetSession` must destroy EVERY collection the bridge writes.** It is a per-file
  list with no compiler help, and a forgotten line is silent: the stale collection
  survives a page reload and reports the previous document's data under the new
  document's session-scoped view ids. Adding a collection means adding a line there.
- **Record subscription keys use a SPACE separator** (`type id`, from the store's
  `REC_SEP = ' '`), not a colon. Split once on the first space: a type can't contain a
  space, a primary key can. `values.js#subscriptionParts` is the single parser and is
  unit-pinned; a colon regression must fail the suite loudly (it silently mis-grouped
  every record key once already).
- **The MAIN-world page hook is unavoidable.** An isolated content script cannot see
  page globals and the bridge cannot see `chrome.*`; `window.postMessage` is the only
  shared channel.
- **Never import `pieces.css`.** The registry theme declares `--color-surface`/`--ink`/
  `--border`, which collide head-on with this panel's own `@theme inline` block and would
  fight the `data-theme` switch panel-glue sets. Alias the pieces' token utilities onto
  `--dt-*` in `styles.css` instead. The `puzzle add piece` installer prints the import as
  an unconditional next step — ignore it here.
- **Pieces `class` props lose to Tailwind's layer ordering.** Compaction needs `!`
  (`py-8!`, `text-[9px]!`). Not author-order — stylesheet order.
- **Flex key/value rows: the value cell needs `flex-1 min-w-0`.** Flexbox's
  `min-width: auto` default makes truncation impossible, and an unbounded key starves the
  value down to its ellipsis. The key is capped (`max-w-[40%] truncate`); full values live
  in `title`.
- **The view tree is a FLAT keyed list.** Depth is `padding-left`; indent guides are a
  clipped repeating gradient, not DOM. Collapse and re-snapshot patch individual rows —
  do not "clean this up" into nested components.
- **jsdom has no Clipboard API**, and CopyButton bails silently without one. Copy
  assertions need the recorder stub in `panel-app.test.js`, or they pass vacuously.
- **Gate commits on vitest's real exit code**, not on grepping its output — grep's exit
  status is about matching, not about tests passing.

## Layout

```
extension/          MV3 package, copied verbatim into dist-extension/
  page-hook.js      MAIN world, document_start: owns the hook, buffers 500 events
  content-script.js ISOLATED relay: postMessage ↔ runtime port
  background.js     service worker: pairs ports by tab id
  panel-glue.js     the ONLY file touching chrome.* from the panel side
panel/              the panel UI — a Puzzle app
  app/bridge.js     the single chrome→store seam; protocol events become records
  app/values.js     pure projections (parsing, previews, diffing) — unit-tested
protocol/           message names + PROTOCOL_VERSION
test/fixture-page/  the synthetic bridge: the permanent protocol test double
```

Panels never call `chrome.*` and never subscribe to the bridge directly. Messages land in
the store via `bridge.js`, which bumps monotonic counters (`connection.viewSeq`,
`flushSeq`, `perfSeq`, `pview.pulseAt`); a subscribed `data()` sees them move and
schedules its own debounced request. This is why a page with no Puzzle app produces zero
traffic — including from the Performance panel, whose polling only ever runs between a
`perf:start` and the report that says recording stopped.

## Registry pieces

Installed with `puzzle add piece <name> --registry <path-to-puzzle-pieces/registry>`;
`pieces.lock` records hashes. **In use:** `split-panel`, `copy-button`, `empty`, `badge`.
**Evaluated and rejected, with reasons worth preserving:**

- `data-table` — no row-click callback and no row identity in the DOM, text-only cells
  (no badges/titles), re-sorts internally even in controlled mode, density unreachable.
- `tree` — single `label` per node: nowhere for the module basename, the flush pulse, the
  hover-highlight, or the trailing log button; `class` reaches only the root `<ul>`. Its
  keyboard model is better than ours; adopt if it grows `meta`, a per-node class hook,
  `@hover`, and a trailing slot.
- `description-list` — no per-item class, `title`, or slot.

Piece sources are never edited. Anything that cannot be adapted through props is a
finding for the `puzzle-pieces` repo.

## Conventions

- Match the surrounding density and comment style; the panel targets DevTools scale
  (10–12px), not app scale.
- Both themes always: every color goes through a `--dt-*` token, and dark theme runs
  darker than Chrome's own because the text is smaller. Contrast ratios are documented
  inline in `styles.css` — keep them ≥7:1 for primary text, ≥4.5:1 for muted tiers, on
  all three background tiers.
- Prefer extending `values.js` (pure, unit-testable) over logic in `.pzl` scripts.
- Structural test pins are fine, but pin the mechanism, not incidental utility classes.

---
> Source: [magic-spells/puzzle-devtools](https://github.com/magic-spells/puzzle-devtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
