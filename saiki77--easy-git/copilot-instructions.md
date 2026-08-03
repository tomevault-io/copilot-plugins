## easy-git

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Easy Git is an **Obsidian community plugin** that syncs individual vault folders to GitHub repos via the GitHub Git Data API — no `git` binary, no shell, mobile-compatible. It targets one or more `(repo, branch, remote-path)` destinations per vault folder, with bidirectional sync, conflict resolution, and push-side Markdown transforms for GitHub rendering.

## Commands

```sh
npm install          # install deps
npm run dev          # watch build (inline source maps, no minify)
npm run build        # type-check then production build (minified main.js)
npm run lint         # ESLint over src/**/*.ts
node version-bump.mjs  # bump version in manifest.json + versions.json
```

`npm run build` runs `tsc --noEmit` first, so type errors block the build. There is no test suite.

## Architecture

### Entry point

`src/main.ts` — `EasyGitPlugin extends Plugin`. Responsible for:
- Loading/migrating/healing settings on startup
- Wiring auto-sync schedules (interval, on-save debounce, startup)
- Dispatching `syncMapping()` calls and tracking in-flight syncs
- Surfacing Notices and the status-bar indicator
- Registering all ribbon/command-palette commands

### Core data types (`src/types.ts`)

- `FolderMapping` — one logical mapping (vault folder ↔ one or more destinations)
- `MappingDestination` — a single `(repoOwner, repoName, branch, remoteFolder)` target; holds its own `lastSyncState` (the SHA map from the last successful sync)
- `PluginSettings` — top-level settings object persisted to Obsidian's `data.json`
- `SyncResult` — per-destination outcome returned from the engine
- `ConflictEntry` / `FileAction` / `FileOp` — classifier output types

### Sync engine (`src/sync/engine.ts` — `SyncEngine`)

Orchestrates one mapping run across all its destinations sequentially. Per destination, the nine-step flow is:

1. Pin remote HEAD (commit SHA + root tree SHA)
2. Walk remote folder → flat `{path → blob SHA}` map
3. Walk local folder → compute git blob SHAs, apply exclusions, rewrite wikilinks in-memory
4. Load `lastSyncState` as the 3-way merge base
5. Classify every path via `src/sync/classifier.ts`
6. Resolve conflicts (mtime auto-resolve → 3-way text merge → modal)
7. Apply pull actions locally (with backups before any overwrite)
8. Build one atomic commit via GitHub's Git Data API and update the ref with non-FF protection (retries up to 3× with 1s/3s/9s backoff)
9. Replace `lastSyncState` with the final SHA map

### Classifier (`src/sync/classifier.ts`)

Stateless function that takes `(localFiles, remoteFiles, lastSyncState, direction)` and returns `FileAction[]` — one of `push-add / push-modify / push-delete / pull-add / pull-modify / pull-delete / noop`. Direction gates which actions are emitted vs. turned into informational notices.

### GitHub layer (`src/github/`)

- `client.ts` — thin `GitHubClient` wrapping `requestUrl` (Obsidian's fetch), shared auth header, rate-limit check, `GitHubApiError`
- `git-data.ts` — Git Data API calls: `createBlob`, `createTree`, `createCommit`, `updateRef`, `getBranchHead`, `listRemoteFolderFiles`, `getBlobContent`
- `auth.ts` — OAuth Device Flow implementation (polling `github.com/login/oauth/access_token`)

### Sync utilities (`src/sync/`)

- `blob-sha.ts` — compute git blob SHA-1 locally (matches `git hash-object`), base64/ArrayBuffer helpers, text-path detection
- `classifier.ts` — 3-way diff table → `FileAction[]`
- `exclusion.ts` — glob matching for the global exclude list + per-mapping `.easygitignore`
- `wikilink-rewrite.ts` — in-memory push-only rewrite of `![[wikilinks]]` → CommonMark; Excalidraw companion resolution
- `markdown-transforms.ts` — push-side transform + pull-side restore for callouts, `==highlights==`, and KaTeX `\phantom` macros. Uses hidden HTML comments for lossless round-trips.
- `commit-message.ts` — template formatter (`{mapping}`, `{datetime}`, `{added}`, etc.)

### UI (`src/ui/`)

- `mapping-modal.ts` — add/edit a mapping (vault folder picker, destination rows, direction, auto-mode)
- `conflict-modal.ts` — per-file keep-local / keep-remote / keep-both resolution
- `sync-log-modal.ts` — view the in-memory 100-entry sync log
- `diagnose-modal.ts` — diagnostic output for troubleshooting
- `pickers.ts` — `SuggestModal` subclasses for mapping/repo/branch selection
- `device-flow-modal.ts` — OAuth Device Flow UI
- `status-bar.ts` — bottom-right aggregate status indicator with 30s ticker
- `confirm-modal.ts`, `button-compat.ts` — utility components

### Secret storage (`src/secret-storage.ts`)

Guarded wrapper around `app.secretStorage` (Obsidian 1.11.4+, OS keystore backed). `manifest.minAppVersion` is 1.6.6, so every entry point feature-checks and falls back to `data.json` on older builds. `writeStoredToken()` returns a boolean rather than throwing, and callers must treat `false` as "keep the token in `data.json`" so a locked keychain can't sign the user out. The API has no delete, so clearing writes an empty string.

The API is reached through a local `AppWithSecretStorage` / `SecretStorageApi` structural type instead of Obsidian's own `App.secretStorage` and `SecretStorage` declarations. Two reasons, and both matter:

1. Obsidian types `App.secretStorage` as always present, so a direct member access type-checks fine while being `undefined` at runtime on every supported build below 1.11.4. Declaring only the shape we use, with `secretStorage?`, keeps the optionality honest.
2. The community-plugin validator rejects `eslint-disable` for `obsidianmd/no-unsupported-api` outright, so suppressing the rule is not an option while `minAppVersion` stays below 1.11.4. Not referencing the flagged declarations is.

Do not "simplify" this back to importing `SecretStorage` from `obsidian`; it re-breaks plugin review. Keep this module free of `console.*` too, for the same reason (the validator warns on ungated logging, and the failure paths already report via return values).

### Settings (`src/settings.ts`)

`EasyGitSettingTab` — renders the collapsible settings UI. Calls `plugin.refreshAutoSyncWiring()` after any change that affects scheduling.

## Key invariants

- **No git binary.** Everything goes through the GitHub REST/Git Data APIs.
- **Vault is never modified by push.** Wikilink and Markdown transforms are in-memory only; the rewritten bytes exist only in the blob payload sent to GitHub.
- **Backups before any local overwrite.** Every `pull-modify` / `pull-delete` snapshots the existing file to `.easy-git-backup/<timestamp>/` first. A backup failure aborts the sync rather than silently overwriting.
- **One atomic commit per destination per run.** If the ref update fails (non-FF), the whole run retries from scratch — never partial commits.
- **`lastSyncState` is the merge base.** Clearing it (via the "Reset sync state" command) forces a full re-scan on the next run, which is safe but will re-surface any divergence in the conflict modal.
- **`healSettings()` runs on every load.** It is idempotent and never deletes a mapping. Broken mappings are flagged in the UI and their Sync button disabled.
- **The token is never written to `data.json` when the OS keystore is available.** `hydrateToken()` and `persistableSettings()` in `main.ts` are the only two places that know where the token physically lives; everything else just reads `settings.auth.token`. Two consequences: all settings writes must go through `saveSettings()` (never `saveData()` directly, or the token leaks back into `data.json`), and `hydrateToken()` must run *before* `healSettings()`, which signs the user out when it sees an auth method with no token.

## Build output

`npm run build` produces `main.js` (bundled CJS, minified). The release workflow at `.github/workflows/release.yml` uploads `main.js`, `manifest.json`, and `styles.css` on tag push. `main.js` is committed to the repo so users can install manually.

## ESLint notes

Two rules from `eslint-plugin-obsidianmd` are disabled intentionally (see `eslint.config.mjs`):
- `obsidianmd/ui/sentence-case` — plugin uses Title Case on buttons/status intentionally
- `obsidianmd/rule-custom-message` — `console.log` is gated behind `debugLogging`; errors are logged only on genuine failures

---
> Source: [Saiki77/Easy-Git](https://github.com/Saiki77/Easy-Git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
