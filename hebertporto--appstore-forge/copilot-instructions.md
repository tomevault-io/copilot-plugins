## appstore-forge

> A local macOS app (Electron + React + canvas) that turns raw app screenshots into

# AppStore Forge

A local macOS app (Electron + React + canvas) that turns raw app screenshots into
store-ready App Store and Google Play assets.

Read `_context/` before changing anything:

- `_context/domain.md` — the vocabulary and the data model. Read first.
- `_context/rules.md` — the invariants. These are not style preferences; breaking
  them produces wrong exported pixels.
- `_context/workflows.md` — how to build, install, version, and verify.

## Build and install

The app is a **local personal build**. It is unsigned and not distributed, so
there is no notarization or auto-update step.

```bash
pnpm install          # once
pnpm install:app      # build → install to /Applications → launch
```

`pnpm install:app` is the command you want almost every time. It runs
`pnpm dist`, removes the old `/Applications/AppStore Forge.app`, copies the new
one in, and opens it.

| Command             | Use it when                                                           |
| ------------------- | --------------------------------------------------------------------- |
| `pnpm install:app`  | You changed something and want it in `/Applications` now              |
| `pnpm dist`         | You only want the `.dmg` in `release/`                                |
| `pnpm app`          | Unpacked `.app` in `release/mac-arm64/`, skips the DMG step — fastest |
| `pnpm electron:dev` | Iterating on the UI, with hot reload                                  |
| `pnpm dev`          | Plain browser version on :4324                                        |
| `pnpm typecheck`    | Before any build                                                      |
| `pnpm lint`         | ESLint                                                                |
| `pnpm test`         | Vitest — pure logic only, see `_context/workflows.md`                 |
| `pnpm format`       | Prettier over the repo                                                |

### Quit the app before repackaging

`electron-builder` deletes `release/mac-arm64/` while building. If the app is
running from there, it gets pulled out from under the running process and dies
with no crash report. Always:

```bash
pkill -f "AppStore Forge.app"; sleep 1; pnpm install:app
```

This also applies to automated UI verification — never repackage while an agent
is driving the app, or you will get phantom crash reports.

## Versioning

`package.json` `version` is the single source of truth. It reaches three places
automatically:

- the macOS bundle (`CFBundleShortVersionString`)
- the DMG filename (`AppStore Forge-<version>-arm64.dmg`)
- the sidebar subtitle, via `__APP_VERSION__` injected by `define` in `vite.config.ts`

```bash
pnpm version patch --no-git-tag-version   # or minor / major
pnpm install:app
```

Never hardcode a version string anywhere else.

## Gatekeeper

The build is unsigned (`identity: null`). Because it is built locally, macOS does
not quarantine it and it opens normally. Copied to another Mac it needs a
right-click → Open once. Do not add signing config unless the app is actually
going to be distributed.

## Verifying a change

The preview is a `<canvas>`. The DOM tells you nothing about whether the drawing
is correct — a `describe` tree cannot see a wrong bezel, a clipped headline, or a
font that silently fell back. Visual changes must be checked from screenshots,
and pixel-level claims must be checked with `window.__renderExport`.

See `_context/workflows.md` for the automation hooks and the standard checks.

---
> Source: [hebertporto/appstore-forge](https://github.com/hebertporto/appstore-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
