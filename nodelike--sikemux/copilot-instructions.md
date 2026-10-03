## sikemux

> - Work directly on the `main` branch for day-to-day work and nightly releases.

# Repository Instructions

## Git workflow

- Work directly on the `main` branch for day-to-day work and nightly releases.
- Release branches are the only exception. Cut `release/<major.minor>` from `main`
  to stabilise a stable release, and cut hotfix branches from `release/*`.
- Never merge a release branch into `main`. Cherry-pick the fix commit instead, so
  version-bump commits stay on the release line.
- Do not create or use Git worktrees.
- Commit day-to-day changes directly to `main`.
- Make small, atomic commits as work progresses.
- Other agents are always working in the same tree at the same time. Stage only the
  files your own session touched, by path, and commit those. Never `git add -A`,
  `git add .`, or `git commit -a`, and never stash, revert, or amend anything you
  did not write. Unrelated dirty files belong to someone else — leave them alone
  and do not mention them as blockers.

- Do not proactively write comments in code. We prefer code to be self explanatory. When we write comments its because there is something locally unintuitive that a future reader should know. But as we write code our goal is to make all code locally intuitive, removing the need for comments. If we ever do need to write comments, we never introduce jargon. Comments should be understandable to someone who was just dropped into the codebase for the first time. Comments should attempt to be concise, on average 1-2 lines. If you are writing a longer comment its likely there is a lot of useless information, which is bad because the information may become stale as the code changes
- Do not leave random markdown files in the codebase that are meant to be some way to deliver information to me. If you want to write a markdown file write it in a temporary file, and give me the path and chat and I can read it
- Never write code that is explicitly backwards compatible. Systems should handle backwards compatibility (like migrations), not logic. If there is some logic that needs to be written otherwise it would appear it would break older users, you MUST make the assumption that no users have ran that code yet and its unreleased, so it would not make sense to consider the side effects that code would produce. This is a safe assumption because the maintainers of this codebase always ensure code that gets shipped is compatbile with the systems that allow for us to not have to explicitly hardcode backwards compatibility

## Releasing

- Follow `docs/releasing.md`. Releases publish only from the Release workflow, started by
  pushing a `v*` tag; never run `scripts/release.sh --publish` yourself, and never approve
  the `release` environment for the owner.
- Nightlies are tagged on `main`. Stable releases and hotfixes are tagged on
  `release/<major.minor>`, and their version bump and notes go only there.
- Never check out `release/*` in this shared checkout. Commit to it from objects with a
  temporary index, as `docs/releasing.md` shows.
- After a stable `0.x.y`, number `main`'s nightlies `0.(x+1).0-nightly.N`.

## Mobile app

- The phone app is in `mobile/` (Expo, `mobile/app`) with the core's Rust client bridged
  in `mobile/native` from `src-tauri/crates/sikemux-mobile`. `mobile/` is its own pnpm
  workspace: never add it to the root install, scripts or checks.
- The phone and the core share `sikemux-core`'s protocol. A protocol change must keep
  `sikemux-mobile` building, and bumps `PROTOCOL_VERSION` once anything already released
  speaks the old shape.
- Phone builds need rustup's Rust first on `PATH`; Homebrew's Rust ignores
  `rust-toolchain.toml` and has no phone targets.
- The bindings `uniffi-bindgen-react-native` generates are build output; do not commit or
  hand-edit them.
- Phone screens are designed in `mobile/design/screens.src.html` before they are built, and
  it must keep matching the app. Change it in the same commit as the screen it draws; run
  `pnpm design` in `mobile/` to view it.

## Server

- The accounts backend is in `server/`: the API (`server/api`), the web app at
  app.sikemux.com (`server/app`), the shared protocol (`server/protocol`) and what runs
  them on citadel (`server/deploy`). It is its own pnpm workspace: never add it to the
  root install, scripts or checks. Run `pnpm check` in `server/`.
- The protocol is generated from `server/protocol/schema`. Change the schema and run
  `pnpm protocol:generate`; the TypeScript, the OpenAPI document and the core's
  `accounts/protocol.rs` are build output that CI compares against it. Within `/v1` the
  schema only grows: never remove or rename a field, route or enum value a released app
  reads.
- Migrations in `server/api/migrations` are never edited once merged. A change that
  removes something comes in two steps: stop using it in one release, drop it in a later
  one, so a rollback always finds the schema it expects.
- Merging to `main` deploys anything under `server/` to production. Do not run
  `server/deploy` scripts against citadel yourself; ask first.

## Website

- sikemux.com is a separate Astro repo, `nodelike/sikemux-front`, checked out at
  `~/projects/personal/sikemux-front`. Pushing its `main` deploys to Vercel.
- The site reads the version, the download link and the download size from GitHub's
  latest release, both at build time and in the visitor's browser. It finds the
  download by the `_aarch64.dmg` suffix, so renaming that asset or leaving it off a
  release breaks every Download button. Nightlies are pre-releases and never show up.
- The README rounds the download to "about 13 MB" and the site says it is smaller than
  Ghostty (33.8 MB). Update both if a release moves the DMG past either.
- Its screenshots come from this repo:
  `pnpm showcase --site ~/projects/personal/sikemux-front/src/assets/shots`.
- Its copy must stay true of the shipped app and its look follows `DESIGN.md`. When a
  change adds, removes or renames something the site describes, say so, so the site
  can be updated.

## UI rules

- Never mark a selected, focused or active item with a coloured bar down its left
  edge (an inset `box-shadow`, `border-left` or pseudo-element stripe). On a rounded
  row the bar bends around the corner and looks broken. Show selection with a tinted
  border and background on the whole item instead.
- Do not mix rounded and square shapes in one group. Every `<button>` gets
  `--radius-2` from `modern-shell.css`, so never put buttons inside a square box
  with dividers between them. Give each item its own border and corner, with a gap
  between items.

---
> Source: [nodelike/sikemux](https://github.com/nodelike/sikemux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
