## infinite-os

> Operating guide for anyone (human or AI agent) changing code in this repo. Pairs with

# AGENTS.md — working agreement for Infinite OS

Operating guide for anyone (human or AI agent) changing code in this repo. Pairs with
[CONTRIBUTING.md](CONTRIBUTING.md) (human-oriented) and [SECURITY.md](SECURITY.md).
If you read one thing: **work on a branch, open a PR, let CI gate it, get a review,
squash-merge.** Never push to `main`.

## What this repo is

**Infinite OS** — a self-hosted, local-first growth-analytics runtime. It connects a
user's data sources (GA4, PostHog, Stripe, Meta, Shopify, X) into a local Postgres
database, keeps them synced, and answers plain-English questions from a governed
metric layer. Everything runs on the user's machine; their data never leaves it.

Pure-TypeScript **pnpm-workspace monorepo**, Node ≥ 20:

| Path | What |
|---|---|
| `apps/cli` | the `infinite` command — operator shell + chat |
| `apps/app` | local HTTP API daemon the CLI talks to |
| `apps/worker` | scheduler + background sync jobs |
| `packages/*` | the engine (`db`, `core`, `config`, `connectors`, `runtime`, `metadata`, `analytical-engine`, `setup`, `llm-controller`, `instrument`) |
| `ui-tui/` | terminal UI (Ink) renderer |

> **`packages/instrument` is the published npm package [`infinite-tag`](https://www.npmjs.com/package/infinite-tag)** (NOT private). It's the founder-run installer that adds the GA4 / PostHog / X tracking tags into the *user's own website repo* (via `npx infinite-tag install` — public keys only, idempotent, reversible). `infinite setup` prints a pre-filled `npx infinite-tag install …` after analytics connect; GA4 also auto-installs it in-process (`packages/setup/src/provisioner.ts` → `import("infinite-tag")`). Don't reinvent tag-install logic — it lives here and is published via `.github/workflows/publish.yml`.

## The shipping workflow

`main` is protected: **no direct pushes, squash-merge only, CI (`ci`) must be green.**
Every change — including a maintainer's own — goes through a PR.

```bash
# 1. sync
git switch main && git pull --ff-only

# 2. branch (or a worktree for parallel work)
git switch -c <type>/<slug>            # type ∈ feat | fix | docs | chore | refactor
#   git worktree add ../io-<slug> -b <type>/<slug>

# 3. build
pnpm install
pnpm -r --if-present build             # or just run ./infinite once (it builds on first run)

# 4. verify — ALL must pass before opening the PR
pnpm typecheck                                 # tsc -b
pnpm test                                      # vitest suite (CI gates on a curated subset — see Tests)
PUBLIC_SURFACE=1 scripts/ci/repo-tripwire.sh   # no secrets / internal files tracked

# 5. PR
gh pr create --base main               # conventional-commit title; say what + why + tests run

# 6. review  → a separate pass reads the diff (CI green is necessary, not sufficient)
# 7. merge   → squash-merge, then delete the branch
```

CI also runs **gitleaks** over tracked content. External (fork) PRs run CI with
restricted permissions, and a maintainer approves the first run for new contributors —
fork PRs never receive repo secrets, and the tripwire degrades gracefully when a
secret-gated check can't run.

### Conventions

- **Conventional commits:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.
- One logical change per PR; keep PRs small and reviewable.
- New behavior ⇒ tests. Changed behavior ⇒ updated docs.
- **Author and review are separate passes.** Don't self-approve a change in the same
  breath you wrote it — have a fresh pass (a reviewer or a second read) check it.

## Versioning & releases

**Distribution model — git checkout, not npm.** `install.sh` clones this repo to
`~/.infinite/app` and drops the `infinite` launcher on the PATH. So "the latest
version" of the runtime is **the tip of your branch on `origin`** — there is *no npm
package to poll* for the CLI/runtime. (The only published npm package in this repo is
`infinite-tag` — see the name-collision warning below — and it is unrelated to runtime
versioning.)

- **The version number lives in the root `package.json`** (`version`), single source of
  truth — `infinite version` reads it. It follows **SemVer** (`MAJOR.MINOR.PATCH`) and
  **only changes when a human bumps it** — nothing auto-bumps it (no changesets, no
  semantic-release). A commit to `main` moves the git SHA; it does **not** move the
  version. Pre-1.0, bump **PATCH** for fixes/small features and **MINOR** for larger ones.
- **Bump as part of the PR that completes the work** — edit `version` in the same PR,
  don't leave it trailing.
- **The on-launch update is SHA-based, not version-based.** The `infinite` launcher
  fast-forwards the checkout on launch (once/24h, clean tree only, ff-only, offline-safe,
  skipped for `GROWTH_OS_CLI_NONINTERACTIVE=1`); opt out with `INFINITE_NO_AUTO_UPDATE=1`.
  `maybeNotifyUpdateAvailable` (in `apps/cli/src/index.ts`) prints the `⬆ Update
  available` notice. Both compare `origin/<branch>` vs local HEAD, so they react to every
  commit on `main`, not to releases.
- **Cutting a release (optional, manual).** A *release* is a blessed commit, marked by a
  **git tag** + a **GitHub Release** (the Release can host assets — e.g. `v0.1.0` hosts
  the GA4 `client_secret`). To cut one after the version-bump PR merges:
  ```bash
  git tag vX.Y.Z && git push origin vX.Y.Z
  gh release create vX.Y.Z --generate-notes
  ```

> [!WARNING]
> **"git tag" ≠ `infinite-tag` — do not confuse them.** A **git tag** (`vX.Y.Z`) is a
> label on a commit in *this* repo that marks a release. **`infinite-tag`** is our
> *published npm package* (`packages/instrument`) that installs analytics tracking tags
> into a **user's own website repo** (`npx infinite-tag install`). Cutting a git tag does
> **not** touch, rebuild, or republish the `infinite-tag` package, and vice-versa. They
> share three letters and nothing else.

## Golden rules (non-negotiable)

1. **Never commit secrets.** `.env*` and `.growth-os/` are gitignored — never
   `git add -f` them. Real secrets live in a local `.env`; `.env.example` documents
   names with placeholders only. The tripwire + gitleaks fail the build on a leak.
2. **Data safety.** Connector credentials, synced rows, and chats live in the user's
   local Postgres + `.growth-os/` — never in git.
3. **`GROWTH_OS_ENCRYPTION_KEY` is load-bearing** — it decrypts stored credentials.
   Don't rename it or change its handling casually; rotating it orphans existing
   connections (forces re-auth).
4. **Don't break the wire.** Some string constants are load-bearing protocol/wire
   values (e.g. provider `User-Agent` strings). Don't "tidy" them without checking
   what reads them on the other end.
5. **Test a `bin` the way users run it.** If you touch a package's `bin` entry, verify
   it via the bin symlink / `npx`, not just `node dist/...` — a direct path can mask
   an entrypoint-resolution bug that only bites under a symlinked launcher.

## Meta Ads writes (money-safety)

Meta Ads campaign/ad-set/creative/ad **writes are operator-only** (the `create_meta_*`
and `set_meta_entity_status` actions are `OPERATOR_ACTIONS` — a `tool_agent`/LLM session
can never fire one) and go over a **direct Graph-API POST** transport. Two rules are
load-bearing and must never regress:

- **Create always lands PAUSED.** Every create hard-codes `status:"PAUSED"`; the input
  types carry no `status`, and an echoed `ACTIVE` is treated as a money-safety violation.
  Creates run **inline** (never enqueued as a retryable worker job) and surface as
  **non-retryable** regardless of status code.
- **Activation is separate and gated.** Going live is a distinct
  `infinite meta <obj> activate <id>` → `set_meta_entity_status status=ACTIVE`, behind a
  **stricter typed-confirm** (type the entity id or `activate`). Creates/pauses use the
  standard `--yes`-skippable confirm; `"meta"` is also in `requiresOperatorConfirmation`.

Never log the access token or raw budget/bid amounts; the audit log records
`budget_present`/`client_token`/resulting-status only. There is no `--token` CLI flag.

## Tests

Single root `vitest.config.ts`. `pnpm test` runs the whole suite; run one file with
`pnpm exec vitest run <path>` — **not** `pnpm --filter`.

**CI is the source of truth, and it gates on a *curated subset* that excludes
`packages/llm-controller`** — some of its tests depend on local model auth / the OS
keychain and flake off a configured machine. So if `pnpm test` fails *only* in
`llm-controller` and is unrelated to your change, that's the known environment
dependence, not a regression — what your PR must turn green is CI.

## Security

Report vulnerabilities privately — see [SECURITY.md](SECURITY.md). Don't open public
issues for them.

---
> Source: [Infinite-Labs-AI/infinite-os](https://github.com/Infinite-Labs-AI/infinite-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
