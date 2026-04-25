## changesets-gitlab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`changesets-gitlab` is a GitLab CI CLI tool that brings [changesets](https://github.com/atlassian/changesets) automation to GitLab, similar to the [GitHub Action](https://github.com/changesets/action). It creates/updates merge requests for version bumps and changelogs, and optionally publishes packages to npm.

Fork of [un-ts/changesets-gitlab](https://github.com/un-ts/changesets-gitlab) by JounQin.

## Commands

```sh
pnpm install          # Install dependencies
pnpm build            # Build (build:r for CJS via @pkgr/rollup + build:ts for ESM+types via tsconfig.lib.json)
pnpm lint             # Lint (runs both lint:tsc = tsc --noEmit AND lint:biome = biome check)
pnpm test             # Run tests (vitest run with istanbul coverage)
pnpm test -- test/utils.spec.ts  # Run a single test file
pnpm cli [command]    # Run CLI locally (uses tsx to run src/cli.ts directly)
```

## Architecture

The CLI has two commands defined in `src/cli.ts`:

- **`comment`** (`src/comment/`) — Runs on MR pipelines (`CI_PIPELINE_SOURCE == "merge_request_event"`). Posts/updates a discussion or note on the MR indicating whether changesets are present, with links to add one. Detects changeset files in `.changeset/*.md` from MR diffs. Handles bot note detection with random GitLab internal username pattern matching (`/^((?:project|group)_\d+_bot\w*)_[\da-z]+$/i`). Supports both `discussion` (threaded, auto-resolvable) and `note` (simple) comment types via `GITLAB_COMMENT_TYPE`. The module is split into focused files: `comment.ts` (orchestrator), `getNoteInfo.ts`, `hasChangesetBeenAdded.ts`, `getAbsentMessage.ts`/`getApproveMessage.ts` (MR comment bodies), `getNewChangesetTemplate.ts`, `getReleasePlanMessage.ts`, `isChangesetBotNote.ts`/`isMrNote.ts` (note detection).

- **`main`** (default, `src/main.ts`) — Runs on the default branch. Accepts `published` and `onlyChangesets` optional inputs. Two paths:
  - **Has changesets** → `runVersion()` in `src/run.ts`: runs `changeset version` (or custom `INPUT_VERSION` script), commits changes, creates/updates a `changeset-release/<branch>` MR.
  - **No changesets + publish script** → `runPublish()` in `src/run.ts`: runs the publish script, parses "New tag:" lines from stdout to detect published packages, pushes git tags, creates GitLab releases from CHANGELOG.md entries.

### Configuration model

Despite not being a GitHub Action, this tool uses `@actions/core` (getInput/setOutput/setFailed) and `@actions/exec` for I/O. `getInput('foo')` reads `process.env.INPUT_FOO` (uppercased, `INPUT_` prefixed). All user configuration comes through `INPUT_*` env vars (e.g., `INPUT_PUBLISH`, `INPUT_VERSION`, `INPUT_TITLE`, `INPUT_COMMIT`) and direct `GITLAB_*` / `CI_*` env vars. See `src/types.ts` for the full `Env` type definition including GitLab CI predefined variables and merge request variables.

### Key modules

- `src/Logger.ts` — Simple logger class with context-based prefix (`[changesets-gitlab:<context>]`) and enable/disable toggle. Used for debug logging across the codebase.
- `src/api.ts` — Creates `@gitbeaker/rest` Gitlab client with proxy support (`global-agent`). Supports personal access tokens and OAuth tokens via `GITLAB_TOKEN_TYPE`. Proxy bootstrap is one-shot (guarded by `bootstrapped` flag). Auto-enables verbose fetch logging when `GITLAB_CHANGESETS_DEBUG` is truthy.
- `src/env.ts` — Merges `process.env` with defaults and dotenv. Uses a getter for lazy `GITLAB_TOKEN` validation — the token is only checked when first accessed, not at startup. `GITLAB_CHANGESETS_DEBUG` is validated via `TRUTHY_VALUES` (accepts `'true'` or `'1'`).
- `src/context.ts` — Exposes `projectId` (`CI_PROJECT_ID`) and `ref` (`CI_COMMIT_REF_NAME`).
- `src/get-changed-packages.ts` — Used by the `comment` command (`src/comment/`). Walks the entire repo tree via filesystem to build a package graph (supports pnpm, yarn, bolt workspaces), parses changeset files from MR diffs, and assembles a full release plan via `@changesets/assemble-release-plan`.
- `src/utils/` — Split into one file per function/constant group. `index.ts` re-exports everything. Contains a **different** `getChangedPackages(cwd, previousVersions)` used by the `main` command — compares current package versions against a pre-version snapshot to find what changed. Also: changelog parsing via remark/mdast (`getChangelogEntry`), `execWithOutput`, `sortTheThings` comparator, `cjsRequire` polyfill, and constants (`TRUTHY_VALUES`, `FALSY_VALUES`, `GITLAB_MAX_TAGS`, `BumpLevels`).
- `src/git-utils.ts` — Git operations (push, reset, branch switching, commit, setupUser).
- `src/read-changeset-state.ts` — Reads changeset state and pre-release mode. Filters out already-released changesets in pre mode.
- `src/types.ts` — `Env` type with discriminated unions for CI vs non-CI and MR vs non-MR contexts. Uses `LooseString<T>` pattern for env vars with known values but allowing overrides.

### Important: two `getChangedPackages` functions

- **`src/get-changed-packages.ts`** — Used by `comment`. Takes MR diff file paths + GitLab API client. Walks the filesystem tree, builds workspace package graph, parses changeset markdown files, assembles a release plan. Returns package names + release plan.
- **`src/utils/getChangedPackages.ts`** — Used by `runVersion` in `run.ts`. Takes `cwd` + a `Map<dir, version>` snapshot. Compares current versions against the snapshot to find packages whose version changed. Returns `Package[]`.

### Tag pushing strategy

`runPublish` in `src/run.ts` uses a `GITLAB_MAX_TAGS` threshold (4). If the workspace has fewer packages than this limit, it pushes all tags at once (`git push --tags`). Otherwise, it checks the `git_push_create_all_pipelines` feature flag via the GitLab API — if active, pushes all; if not, pushes tags individually per released package to avoid triggering excessive pipelines.

### Design notes

- Dual ESM/CJS output: ESM is the primary format (built by `tsc` via `tsconfig.lib.json`), CJS is generated by `@pkgr/rollup` (`build:r`).
- `global.d.ts` declares the `GLOBAL_AGENT` global used for proxy configuration and `vitest/globals` reference.
- TypeScript strict mode with `verbatimModuleSyntax` — use `import type` for type-only imports.
- Biome (`biome.json`, extends `@dvashim/biome-config/react-balanced`) handles formatting and linting alongside TypeScript's type checking.
- Tests live in `test/` (not co-located in `src/`). Vitest with `globals: true` — no need to import `describe`/`test`/`expect`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvashim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
