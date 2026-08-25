## spotless

> > Read this before changing anything. The conventions below are carried over

# Spotless, root agent rules

> Read this before changing anything. The conventions below are carried over
> from `~/code/v2` and adapted: that repo is a TypeScript monorepo, this is a
> Native SDK app, so the *discipline* transfers and the *specifics* do not.
> Copying v2's stack rules verbatim here would be the exact slop they exist to
> prevent.

## Mission

Spotless files new downloads into subfolders, by extension and by where the
file was downloaded from. It is a **single-folder** tool: it reads one
directory, moves files within it, and never deletes anything.

## The dependency arrows (enforced by the compiler, not a script)

```
src/shared.ts      ← core + services   (boundary records only; imports nothing)
src/bytes.ts       ← core              (byte primitives)
src/rules.ts       ← core              (the rule table + pure helpers)
src/editing.ts     ← core              (the text-field reducer)
src/core.ts        ← the views          (Model, Msg, update, derived bindings)
src/services/*.ts  ← NOBODY            (reached only through @native-sdk/services)
```

The core **never** imports a service, even `import type`. That is NS1065 and
the checker stops the build. Services may import subset-legal declarations from
the core class; every record that crosses the boundary is declared once in
`src/shared.ts` and nowhere else.

`update` is pure and synchronous. It never touches the filesystem: it returns a
`Cmd` and the runtime dispatches the result back as a `Msg`.

## Naming

- `.ts` files kebab-case; `.native` view files named for their window label
  (`src/windows/settings.native` is the window whose label is `settings`).
- Model fields and Msg kinds keep their TypeScript spellings, because markup
  binds them verbatim. A rename is a two-file change.
- `Msg` kinds are `snake_case` verbs (`toggle_rule`, `pick_folder`); model
  fields are `camelCase` nouns.
- Derived state is a helper, never a field. If markup needs it, export a
  single-Model function.

## Comments policy

`///` doc comments and `//` comments that explain **why**. No `====` banners,
no narration of what the next line does. A comment that restates the code is
deleted, not reworded.

## Type safety

- The core is the app-core subset: no `any`, no `as` beyond a narrowed catch,
  no ambient IO. The checker enforces this. Read the NS**** diagnostic, it
  names the fix and the reason.
- Integer model slots must be proven whole **at the assignment**, and the proof
  does not survive a local or a helper's return. See the README.
- Services are ordinary TypeScript but the *static* tier: consult
  `native skills get ts-services` before reaching for a Node API, and expect
  `spawnSync`-style call forms to be narrower than Node's.

## Don't redefine, check first

Before writing a byte helper, read `src/bytes.ts`. Before writing a rule
helper, read `src/rules.ts`. Before formatting anything for display, ask
whether the **service** should format it. It has real TypeScript and the core
does not, which is why sizes, ages and timestamps arrive as bytes.

## Do not add things

- NO new dependencies. The app ships four files of truth and a service; the
  binary carries no runtime. "Convenient" is not a reason.
- NO new top-level folders. `src/`, `src/services/`, `src/windows/`, `assets/`,
  `docs/`, `scripts/`, `.github/` is the whole layout.
- NO abstractions for single consumers: no helper module used by one file, no
  parameter that has one caller, no "future-proofing" for Linux or Windows
  while the app is macOS-only.
- NO scaffolding beyond the change at hand. Finishing early means raising
  quality, verifying with the automation harness or tightening copy, never
  widening scope.
- NO second sorting mechanism. Extension rules and origin rules are the two,
  and they compose through one planner in the service. A third would need a
  reason written down here first.

## Verify with your own eyes

`native check --strict` and `native test` are the floor, not the ceiling. Both
pass on code that renders wrong, because markup errors are visual:

```sh
native build -Dautomation=true
./zig-out/bin/spotless &
native automate wait
native automate screenshot main-canvas 2   # then LOOK at it
```

Every UI bug in this repo's history was found by looking at a screenshot, not
by a green check: an attribute that rendered `{r.ext}` literally, a list that
collapsed to zero height, a virtual extent that let rows bleed past their
region. The checker cannot see any of those.

**Never point it at a real Downloads folder to test.** Use a scratch directory,
confirm the dashboard header shows that path, and only then press Sort Now.
This app does exactly what it is told.

## The cut list (append, never delete)

Linux and Windows support · signing and notarization · Sparkle-style
auto-update · a rules import/export format · per-folder rule sets · undo of a
completed pass · content-based classification (reading file contents) ·
duplicate detection · a menu-bar-only mode with no main window · iCloud or
network volume support · scheduled quiet hours.

FSEvents-driven watching is **deferred, not cut**: it needs a Zig
platform-services extension posting into `Cmd.channelOpen`, and Watch mode's
directory-signature poll is the standing substitute. Anyone reaching for it
should read the README note first.

## Commits and pull requests

Conventional commits, enforced on the `commit-msg` hook by
`commitlint.config.ts`. The hook is wired by `npm install`; `--no-verify` skips
it.

```
git commit -m "feat(rules): match origin rules by parent domain"
```

An optional `ABC-123 ` ticket prefix is part of the grammar but is never
required. No tracker is wired to this repo.

## Branches and CI

- `main` is the trunk. Any other branch pushed to the remote gets a draft PR
  opened against `main` automatically.
- `release` is the release channel. A merge into it builds, packages, and
  publishes a GitHub Release for the version in `app.json`.
- CI runs `native check --strict` and `native test` on every PR and on `main`.
  Bump `app.json`'s `persist.version` whenever the `Model` shape changes. The
  checker warns about it, and `--strict` turns that warning into a failure.

---
> Source: [ripgrim/spotless](https://github.com/ripgrim/spotless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
