## opencode-status-line

> OpenCode v2 TUI plugin: a prompt-footer status line (context, cache, streaming

# opencode-status-line

OpenCode v2 TUI plugin: a prompt-footer status line (context, cache, streaming
speed, cost, elapsed time, uncommitted changes). `README.md` is the friendly
introduction; `MANUAL.md` is the exhaustive user reference.

## What is unusual here

- **No build step.** `package.json` publishes the source (`./tui` →
  `src/tui.tsx`) and OpenCode transpiles it on load; don't add a bundler, and
  keep `src/` the only shipped surface — `bun.lock` exists solely so the
  typecheck job resolves the host and Solid types reproducibly. `bun test` (or
  `bun test test/rate.test.ts` for one module) needs no install;
  `bun install && bun run typecheck` checks types, including `src/tui.tsx`,
  which no test imports; `npm run check:pack` checks the package surface. CI is
  `.github/workflows/ci.yml`; releases are `.github/workflows/publish.yml`, not
  a laptop. `main` takes pull requests only: green CI plus one approving
  review (the maintainer bypasses for their own work).
- **`src/tui.tsx` is the plugin entry**, loaded straight from this checkout —
  the live host's `~/.config/opencode/cli.json` lists the directory. OpenCode
  transpiles the TSX on load and hot-reloads on save, so a broken save shows up
  in the running TUI immediately. The plugin cannot run standalone; verify
  runtime changes by hand in a session (`/opencode-status-line` opens the stats
  dialog).
- **The root `tui.tsx` is a load-bearing shim** re-exporting `src/tui.tsx`. The
  running 2.0.16 loader resolves a directory plugin through `<dir>/tui` before
  checking `package.json` exports; deleting the shim drops the plugin from the
  live TUI. npm consumers resolve `@rashidrazak/opencode-status-line/tui` through exports to
  `src/tui.tsx` instead.
- Internal imports carry `.ts`/`.tsx` extensions (`./rate.ts`); the host
  resolves them verbatim, so keep that style.

## Layout

| Path | Role |
| --- | --- |
| `src/tui.tsx` | Entry: event wiring, slot render, command. The only file importing `@opencode/plugin`, `solid-js`, or host APIs. |
| `src/rate.ts` | Speed maths (sliding window, turn fold, calibration, history) and the `USAGE_LABELS` icon/word sets. Pure. |
| `src/render.ts` | Gauge and context-bar geometry, run cutting and wrapping for narrow widths. Pure. |
| `src/format.ts` | Token / money / duration formatting. Pure. |
| `src/diff.ts` | Uncommitted-change totals from the host's VCS status, and the diff segment's cache policy. Pure. |
| `src/palette.ts` | The bundled colour palettes and palette/override resolution. Pure. |
| `src/config.ts` | JSON config loader; pure except an injectable `read`. |
| `test/*.test.ts` | One per pure module. |
| `tui.tsx` | Root shim re-exporting `src/tui.tsx`; see above. |

Keep new logic in the pure modules so it can be tested without a terminal.

## Host-API traps (each cost a TUI restart to learn)

### Entry rules — every edit to `src/tui.tsx`

- Register the keymap layer inside the `app` slot's `render`, never directly in
  `setup`: v2 keeps the keymap provider in the component tree, so a `setup`
  registration throws `Keymap.Provider is missing` and kills the plugin. Give
  the layer `mode: "global"`: a layer that names no mode is pinned to `base`,
  and v2 pushes `autocomplete` while the slash list is open and `modal` while a
  dialog is, so a mode-less command is unreachable in the two places it would
  be found.
- Build rendered parts inside a `createMemo`. `Show` calls its children
  untracked, so a plain array is evaluated once and the line never repaints.
- Route every event handler through `safely`; an uncaught throw inside one can
  kill the plugin generation, and a half-saved file has done exactly that.
- Saving any `src/` file the entry imports hot-reloads the plugin: the module is
  re-imported and module scope comes back empty, which used to blank the meter
  segment mid-turn on every save. State that must outlive a generation lives on
  `globalThis` (`sharedMeters` in `src/tui.tsx`). Touching `README.md`,
  `MANUAL.md` or `test/` does not reload; the `src/` imports do.
- A 250 ms ticker repaints only while a stream is active; a 1 s heartbeat keeps
  the elapsed timer and held figures repainting when nothing streams. Stop
  both in the cleanup function.

### Speed maths and the session record

- The meters are process-scoped: a session met without one — a resume, or a
  reload before any delta — seeds its last figure from the stored messages
  (`seedMeter` in `src/tui.tsx`; `recordedSteps` / `restoreFinal` in
  `src/rate.ts` hold the logic and its rationale). The seed is lazy — the meter
  branch in `usageRows` kicks it when the map misses — and retries on later
  paints, because the host hydrates messages page by page and a partial page
  must not fold as a turn; one `message.sync` per session forces the full fetch.
  It sets `final` plus a resting zero `sliding` (empty gauge, `↯ 0.0`) — never
  `turn`, which a later step would absorb — and the rebuilt figure is close to,
  not bit-identical with, the live one: accept the ~1% tolerance rather than
  chase it with tool-time heuristics. `time.streamed` is a stream-finalisation
  stamp, not a first token (see `firstTokenAt`). Window samples and the
  statistics are memory-only.
- The session record (`data.session.get`) holds token totals cumulative across
  all turns. Context and cache must read the newest assistant message's own
  `tokens` (`windowInfo` in `src/tui.tsx`), or every prompt ever sent is counted.
- Exact token counts arrive only at `session.step.ended`; live figures are
  estimates from stream deltas, calibrated at that point. Prefer the server's
  `event.created` clock for step spans and fall back to local arrival times —
  never mix the two (see `endStep` in `src/rate.ts`). Tool-argument deltas
  (`session.tool.input.delta`) count as output, and the decode span starts at
  the first token, so TTFT is not charged.

### Host registries and the turn lifecycle

- Shell counts come from the host's shell registry (`context.data.shell`),
  which holds a shell only while it executes; background shells live in a
  separate registry and never appear there. Match `status === "running"` and
  `metadata.sessionID`.
- The diff counter reads the host's VCS registry too (`context.client.vcs.status`,
  the working tree against the location's base, untracked files included — its
  route describes itself as "uncommitted working-copy changes"), rather than
  spawning `git` from the TUI. The host answers each request from scratch, so
  the reading is cached per location and re-asked at `diff.refreshMs`, and a
  closing turn marks it stale so the next paint re-asks. A location that cannot
  answer — no repository, no provider — caches as a clean tree; the host caps
  its untracked stat read at 4 KiB, so a large new file can land at zero lines,
  and that is the host's figure, not the plugin's to invent around.
- `session.idle` also closes a turn as a late belt; `endTurn` is idempotent, so
  double-closing is safe.

### Surfaces and width

- `config.surface` is a list of placements: `setup` registers one slot renderer
  per entry (`renderFor(surface)`), each closing over its own
  `stackFor`/`resolvedPadding`/`sharesHostRow` facts and its own `segmentsFor`
  list, owning its own measured-width signal, while the shared `version` signal
  keeps every placement repainting together. The stats command's `app` layer is
  registered once, independent of the list.
- The line renders only when a session is on screen: the renderer resolves its
  session from the slot input or the route, and a non-session route (`home`, a
  plugin page) yields an empty row. `app` and `home.footer.status` are mounted
  by the host on those routes, so the line stands down there by choice — it
  describes a conversation, and none is open. Do not "fix" this with a
  most-recent-session fallback: it was tried once, looked out of place on home,
  and was reverted.
- A `sidebar.*` surface is a narrow column: `stackFor` stacks the segments one
  per row and each row is cut to `columnWidth(context.renderer.width)` with an
  ellipsis. `context.renderer` is the shared OpenTUI renderer, so read its width
  inside the render memo — the window resizes under the line.
- `app` is the window's bottom row: `paddingFor` gives it the composer's 2-column
  indent, a right margin and two clear rows underneath (each side overridable
  per surface through the `padding` config). The host sizes the slot to its
  content — `padding.bottom` already proves that — so the line adds a row
  rather than being clipped.
- A one-line surface fits itself to the width its box was **dealt by layout**,
  not `context.renderer.width`: a footer row shares its width with OpenCode's
  own status text, so the renderer overstates ours. `onSizeChange` reports the
  box's border-box width after every layout pass; store it in a signal (defer
  the write with `queueMicrotask` — the handler runs inside layout) and
  `wrapRows` moves segments that do not fit whole to further rows, with no cap;
  only a segment wider than the line is cut.
- A box in a host row (`sharesHostRow`) pins its **unwrapped** width as its
  `flexBasis`. A row child's width otherwise derives from its own drawn
  content, so once the line wrapped, the box's basis was the wrapped width:
  the shrink deal kept shrinking it and widening the window could never
  restore the line. With the basis pinned, the deal does not move, the
  measurement settles in one pass, and a widened row gets the full line back.
  `app`, the composer top and the sidebars stretch to the host's width and
  need no basis. Footers and sidebars are placed by the host and take no
  padding by default.
- The box also pins its drawn height as its `minHeight` (rows plus padding,
  which is part of the border box). The host mounts `app` as the last child of
  a column beside a transcript that often overflows it, and Yoga's default
  shrink then squashed the wrapped box below its content: every row landed on
  the same line, drawn over one another. The floor makes the transcript absorb
  the shrink instead; wrapping stays stacked and the padding rows survive.

## Config

Precedence: `~/.config/opencode/opencode-status-line.json` (honours
`XDG_CONFIG_HOME`) → `<project>/.opencode-status-line.json` → plugin entry
options. Invalid files or values warn and are ignored, never fatal. Adding a
key means `Config` + `DEFAULT_CONFIG` + validation in `src/config.ts`, plus
`rateOptions` if it is maths, plus MANUAL.md — the key's chapter and
`All settings at a glance`. README stays introductory; touch it only if the
quick-start example or the feature list changed. User-visible changes also ride
under `## [Unreleased]` in `CHANGELOG.md`: each release takes that version's
section as its GitHub Release body (`scripts/changelog-section.mjs`), and the
release fails before publishing without it. `test/config.test.ts` injects
a fake `read`; never touch disk from a test.
`colors.palette` resolves through `src/palette.ts`: a family name follows
`context.themeMode` (the host's resolved `dark`/`light`, never `system`), and a
chosen palette's `muted` ink is where held figures and bar tracks go.
`colors.exclude` marks a segment's runs with `hostRuns` in `usageRows`, which
`toneColor` reads to draw from the theme tokens instead.
`usage.surfaces` overrides `usage.segments` per placement; `segmentsFor`
resolves the fallback, an empty override hides that placement, and the map
merges across config sources like `padding`.

## Publishing

`npm run check:pack` inspects the tarball — every tracked `src/` file packed,
nothing untracked in, no root shim, every `exports` target present — and CI
runs it on every pull request. `.github/workflows/publish.yml` runs on a pushed
`v*` tag and on a published Release: it publishes with npm trusted publishing
(OIDC, `id-token: write`) — no `NPM_TOKEN`, provenance automatic — and creates
the GitHub Release from the version's `CHANGELOG.md` section, checked before
publishing. Every step checks instead of assuming (the tagged commit on `main`,
tag against `package.json`, changelog section, version on the registry,
existing Release), so both events are safe and a release is just `npm version`
plus `git push --follow-tags`. Tags are created only by hand — nothing tags on
merge — the gate just refuses a tag that points off `main`. The one-time npm
setup (hand-published bootstrap, trusted-publisher fields) lives in
`RELEASING.md`. Don't add a publish token unless OIDC is abandoned.

The package is published as source, so `files` in `package.json` carries `src/`
wholesale — every module `src/tui.tsx` imports must be under it, or installs
break. The root shim stays out of the tarball; npm resolution goes through
`exports`. `@opencode/plugin` is the dependency; `@opentui/core`,
`@opentui/solid`, and `solid-js` are peers OpenCode provides. The plugin is
CLI-only, so consumers add the package name to `cli.json`, never
`opencode.json`. README links to files outside the tarball (`MANUAL.md`,
`CONTRIBUTING.md`, `RELEASING.md`) must be absolute GitHub URLs — npm renders
the README with none of the repository's files around it.

---
> Source: [rashidrazak/opencode-status-line](https://github.com/rashidrazak/opencode-status-line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
