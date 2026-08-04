## druk

> Instructions for AI agents working on **druk**, a terminal code editor.

# AGENTS.md

Instructions for AI agents working on **druk**, a terminal code editor.

`CLAUDE.md` is a symlink to this file — keep everything in here.

## What this project is

A TUI code editor built on [OpenTUI](https://github.com/anomalyco/opentui) (Solid
reconciler on a native Zig core). Shipped as a standalone binary — npm, Homebrew, a curl
installer — and run as a CLI.

Features: file tree with bulk file operations, preview/pinned tabs, tree-sitter syntax
highlighting, search (current file and project-wide), command palette, themes, vim mode,
git marks in tree/gutter/status bar, file watching with conflict prompts, per-project
session restore, and a startup update check.

## Runtime and tooling

- **Bun is required to develop** — OpenTUI's native core loads through Bun's FFI. Node
  cannot start the app from source (its `node:ffi` is not in any shipping release), so
  never "fix" a Bun dependency by switching the runtime. Users need nothing installed:
  `bun build --compile` bakes the Bun runtime, the native library and every grammar into
  one executable.
- **bun manages dependencies and scripts.** Do not use npm or pnpm for installs — the
  lockfile is `bun.lock`.
- **Say `bun run <script>`, not `bun <script>`.** `build` collides with Bun's own bundler
  subcommand, so `bun build` silently bundles nothing instead of running the script. This
  now includes `test`: bare `bun test` works, but the whole suite runs in one process and
  takes four times as long — `--parallel` lives in the script and cannot be set from
  `bunfig.toml` (the key is accepted there and silently ignored).

```bash
bun install
bun run start            # run from source, opens the current directory
bun run start ./some/dir # run from source against a directory
bun run build            # compile a binary for this machine into dist/<target>/
./dist/*/druk .          # run what you just built (bin/druk.js finds it too)
bun run build linux-x64  # …or for a named target, if its native package is installed
bun run release          # package dist/ for npm + release archives (--publish to ship)
bun run formula          # Homebrew formula for those archives (not published anywhere yet)
bun run test             # unit + UI, one worker per core (~20s; 87s without --parallel)
bun test test/foo.tsx    # a single file, where the flag buys nothing
bun run check-types      # tsc --noEmit
bun run lint             # oxlint
bun run format           # oxfmt (writes); format:check to verify
```

Always run `bun run check-types`, `bun run lint`, `bun run format` and `bun run test`
before considering a change done — `bun run check` is all four.

`--parallel` runs each *file* in its own worker process, so nothing may depend on state
shared between files. `test/setup.ts` is preloaded to give every worker its own
`XDG_CONFIG_HOME`; without it the workers fight over one `sessions.json` — and the suite
writes to your real `~/.config/druk`.

## Shipping

`bun run build` produces one executable; `bun run release` turns the executables in
`dist/` into npm packages and release archives. Five things about that are easy to break:

- **Assets must be static `with { type: 'file' }` imports.** Bun embeds only what it can
  see at build time, so a computed specifier or an `import.meta.resolve` call leaves the
  binary without that file. Every grammar and query goes through
  `src/languages/grammars.ts` for this reason.
- **The binary must not autoload `bunfig.toml`.** druk is opened inside other people's
  projects, and a standalone Bun binary otherwise reads the `bunfig.toml` it finds there —
  whose `preload` fails to resolve and kills startup. `build.ts` turns that off.
- **Cross-compiling needs the target's `@opentui/core-<platform>` package**, and
  `bun install` fetches the host's alone. That is why the release workflow uses one native
  runner per platform instead of five `--target` flags on one machine.
- **The GitHub release is uploaded before npm.** One package is published, `druk`, and it
  holds no binary: `bin/binary.mjs` fetches the archive for the machine from the release.
  Publishing npm first would leave a window where an install finds no asset.
- **There is deliberately no package per platform.** That is the usual arrangement, and
  it is what druk used to do, but creating a package needs a credential that can create
  packages — while the release authenticates as GitHub through OIDC and may only publish
  to `druk` itself. One package is what makes the release run unattended.

The repo's own `package.json` is `private`: what npm publishes is staged into
`dist/npm/druk` by `scripts/release.ts` — the shim, the postinstall and nothing else.
Versions come from `package.json` — bump it and `.github/workflows/release.yml` builds
every platform, uploads the archives to the release and publishes to npm, with no manual
step. Two ways to start it: push a tag `v<version>`, or run the workflow from the Actions
tab, which tags the commit it runs on for you.

**`package.json` is the version, not the ref.** The published shim fetches its binaries
from `releases/download/v<version>`, so the release must carry exactly that tag — the
workflow reads the version once in `check` and every later step uses it. A tag push whose
name disagrees with `package.json` fails there, before five runners have built.

**The tag is pushed with git, not created by `gh`.** GITHUB_TOKEN may create a release
for a tag that exists, but `gh release create --target <sha>`, which has to create the
tag as well, comes back `403 Resource not accessible by integration` — with
`contents: write` granted and no ruleset in the way. Pushing the tag over the checkout's
credentials first is ordinary `contents: write` and works, so a manual run tags the
commit in its own step and `gh` only ever sees a tag that is already there.

**Every run ships, and both publishing steps go together.** There is no dry run: neither
step may be made conditional on its own, because druk 1.0.0 reached npm from a run whose
release upload was skipped, and the published shim spent its life fetching a release that
did not exist. Re-running a shipped version is safe — `release.ts` skips a version already
on the registry and the upload clobbers its assets.

Homebrew is not wired up yet. `scripts/formula.ts` generates a working formula from the
archives in `dist/release/`, but nothing publishes it: that needs a `letstri/homebrew-tap`
repository and a `TAP_TOKEN` secret, then a step in the release workflow to commit the
formula there.

## Architecture

Read [ARCHITECTURE.md](ARCHITECTURE.md) first. It has the folder map, the one-way
dependency rule, and recipes for the extension points:

| Want to add a… | Edit |
| --- | --- |
| language | `src/languages/grammars.ts` + a query in `src/languages/queries/`, then `src/languages/index.ts` |
| theme | new file in `src/themes/` + register in `src/themes/index.ts` |
| setting | `src/core/config.ts` (`Config`, `DEFAULTS`, `parse`) |
| command | `src/app/commands.ts` + implement the action in `src/app/App.tsx` |
| keybinding | handler in `src/app/App.tsx` or `src/ui/EditorPane.tsx`, advertised in `src/ui/keys.ts` (feeds the footer hints, help overlay and Alt+/ peek) |

`src/app/commands.ts` is the feature index — read it to learn what the editor can do.

`ui/` and the feature folders (`core/`, `languages/`, `themes/`, `editor/`) must never
import from `app/`. State lives in `App.tsx` and flows down as props.

## Rules

### Comments

The bar is high: write a comment only when its absence would let someone break the code.
Assume the reader is competent and can read TypeScript — they don't need the "what", only
the "why you can't do the obvious thing".

Ask: **if I delete this comment, will the next person make a mistake?** If no, delete it.

Worth writing:

- A trap that will be "cleaned up" and reintroduce a bug — non-obvious ordering, a guard
  that looks redundant, a workaround for upstream behaviour.
- A convention the types don't carry — units, offset bases, which coordinate space a
  number lives in.
- An invariant two distant pieces of code silently depend on.

Not worth writing: restating the line below, naming a section, labelling parameters,
explaining a well-named function, TODOs, commented-out code.

```ts
// Bad — restates the code
// increment the counter
count++

// Bad — the signature already says this
/** Saves the file to disk. */
function saveFile(path: string, content: string) {}

// Good — deleting this comment invites a "simplification" that breaks every file
// highlightOnce returns absolute string offsets, but the edit buffer indexes
// text with newlines removed; without this every line drifts one column right.
```

Prefer making the comment unnecessary: a clearer name, a named constant, or a small
function usually beats a sentence explaining the mess.

### Keep this file current

When a change alters how someone works with the project — new script, new dependency,
new extension point, changed layout, changed workflow, a new rule or convention — update
`AGENTS.md` (and `ARCHITECTURE.md` when the structure moves) **in the same change**.
A stale agent file is worse than none.

### Verify behaviour, don't assume it

This is a TUI: type errors do not prove it works. Write a test — `bun test` renders the
real app off-screen and gives you the frame as text, so UI is assertable.

```tsx
const t = await launch(fixture({ 'a.ts': 'const a = 1\n' }))
await press(t, i => i.pressEnter())          // opens the file
expect(t.captureCharFrame()).toContain('const a = 1')
```

`test/helpers.tsx` has `fixture()` (temp project), `launch()` (renders `<App/>`, and takes
a config and a terminal size), `press()`, `settle()`, `pressEscape()` and `runCommand()`.
Highlight helpers live in `test/syntax.ts` instead — `parseHighlights()` and
`allSegments()` — so a unit test can use them without pulling in `<App/>`. Two rules the
harness exists to encode:

- **Yield before capturing.** The reconciler flushes on a macrotask; a frame captured
  straight after a key still shows the previous state. `press()`/`settle()` handle it.
- **Escape needs a gap.** Esc is the prefix of every arrow/function-key sequence, so the
  parser holds it until it knows nothing follows. Use `pressEscape()`, not
  `mockInput.pressEscape()`.

`captureCharFrame()` returns text only — selection and focus are background colors, so
assert on something textual (a prompt appearing, the status bar, file contents on disk).

For a real end-to-end check, drive the built CLI in a PTY with an isolated
`XDG_CONFIG_HOME` so it never writes your real config.

### Solid, not React

Solid compiles JSX at build time and has no re-render — components run once and
signals update the terminal directly. Three rules follow:

- **Never destructure props.** `function X({ a })` freezes `a`; use `props.a`.
- Signals are functions: `count()` to read, `setCount(v)` to write. Derived values are
  `createMemo`; side effects are `createEffect(on(...))` / `onMount` / `onCleanup`.
- Lists need `<For each={...}>` and conditionals `<Show when={...}>` — a bare `.map()`
  or `&&` renders once and never updates.
- Shared mutable state must be a signal or store. A plain exported object (the theme
  palette, for one) updates in memory but repaints nothing.

The Solid transform is a Babel step, so it needs `bunfig.toml` preload entries for both
the app **and** `[test]`, and the build goes through `Bun.build` with
`@opentui/solid/bun-plugin` (tsdown/rolldown cannot do it).

Some OpenTUI element names are snake_case (`line_number` is the one druk uses).

### Style

- TypeScript strict; no `any` escapes without a reason.
- Prefer the smallest change that fits the surrounding code; match its idiom.
- Formatting and lint are enforced by oxfmt/oxlint — run them rather than hand-aligning.
- Keep modules focused; if a file is becoming a grab bag, split it along feature lines.

### Scope

- Do not add dependencies for things the standard library or OpenTUI already does.
- Do not commit or push unless asked.
- Do not edit `dist/` — it is generated and gitignored.

---
> Source: [letstri/druk](https://github.com/letstri/druk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
