## vim-prettier

> This repository has not had sustained maintenance, so do not assume the current

# AGENTS.md

## Compatibility Revival Spec

This repository has not had sustained maintenance, so do not assume the current
`package.json`, lockfile, Docker image, or docs accurately describe supported
Prettier, Vim, Neovim, Node, or parser-plugin versions.

The immediate goal is to turn compatibility from guesswork into a tracked,
tested support policy. Prefer small, evidence-backed changes over broad
modernization.

## Operating Rules

- Work on `dev/compatibility-revival` for this effort unless directed otherwise.
- Keep runtime fixes small and separately reviewable.
- Do not silently drop legacy support. If support is removed, document the reason
  and provide migration notes.
- Treat async formatting as data-loss-sensitive. Add tests before changing async
  buffer replacement, write behavior, or buffer-switch handling.
- Preserve project-local Prettier behavior. A user's project-local Prettier and
  config should take precedence over bundled fallback tooling.
- Do not use dependency upgrades as proof of compatibility. Every Prettier/plugin
  major bump needs fixture coverage.
- Keep README and `doc/prettier.txt` in sync when user-facing behavior changes.

## Proposed Support Matrix

Use this as the starting hypothesis, not as a final promise.

### Blocking Editor Targets

- Vim 8.2 latest patch with `+job` and `+channel`.
- Vim 9.1 or latest stable Vim.
- Neovim 0.9 latest patch.
- Neovim 0.10 or latest stable Neovim.

### Discovery-Only Editor Targets

- Vim 7.4.
- Vim 8.0 and 8.1.
- Neovim 0.4.x.

These should not be advertised as supported unless CI proves they work. The code
already uses APIs and method-call syntax that are unlikely to be Vim 7 safe.

### Blocking Prettier Targets

- Prettier 3 latest as the primary target.
- Prettier 3.0.3 to validate the current 1.0.0-era package baseline.
- Prettier 2.8.8 as a project-local legacy target.

### Discovery-Only Prettier Targets

- Prettier 1.x. Prefer the existing `release/0.x` path for users that need this
  unless a maintainer explicitly chooses to restore active support.

### Language Coverage

Core Prettier languages to test across Prettier 2.8.8 and 3.x:

- JavaScript, JSX, MJS, CJS.
- TypeScript and TSX.
- CSS, SCSS, Less.
- JSON and YAML.
- HTML and Vue.
- GraphQL.
- Markdown and MDX.

External plugin languages to validate primarily against Prettier 3-compatible
plugin versions:

- PHP.
- Ruby.
- XML.
- Lua.
- Svelte.

## Known Baseline Findings

- `yarn install --frozen-lockfile` succeeds, but current PHP and Svelte parser
  plugin versions report Prettier 1/2 peer compatibility while the package ships
  Prettier 3.0.3.
- `yarn test` on Vim 9.1 originally failed all formatting tests with
  `E930: Cannot use :redir inside execute()`, caused by version detection in
  `autoload/prettier/resolver/config.vim` calling `prettier#PrettierCli()` under
  `redir`.
- `yarn lint` now uses pinned local Python requirements and a checkout-local
  virtualenv wrapper for `vim-vint`, rather than relying on global `vint`.
- No GitHub Actions workflow currently defines a compatibility matrix.
- The Dockerfile is old and uses Alpine 3.8 plus unpinned `testbed/vim:latest`.
- Docs and runtime filetype support have drifted.

## Current Branch Progress

- Fixed the Vim 9 `redir` failure by making Prettier CLI version detection call
  the resolved executable directly instead of routing through `:PrettierCli`
  inside `redir`.
- Added a focused vim-driver regression test for config version detection inside
  `execute()`.
- Made the Jest/Vim harness use `tests/vimrc` with `-u NONE` so local tests load
  this checkout instead of a user's installed vim-prettier runtime.
- After those fixes, `yarn test` on local Vim 9.1 reaches formatting assertions:
  15 tests pass and 16 fail.
- Remaining core snapshot failures are compatibility deltas rather than the
  original Vim 9 harness failure: GraphQL, SCSS, YAML, and Vue differ from the
  stored snapshots under the current Prettier 3.0.3 toolchain.
- Remaining external plugin failures are no-op formatting for Lua, PHP, Ruby,
  and XML, consistent with stale or unresolved bundled parser plugins.
- Added an initial GitHub Actions smoke workflow. It runs the Vim 9 config
  version regression as a blocking check and runs the full formatting suite as
  non-blocking discovery until plugin compatibility is restored.
- Updated targeted Prettier 3.0.3 snapshots for GraphQL, SCSS, Vue, and YAML
  after confirming those were core formatter deltas.
- Added explicit `plugins` CLI config support and scoped automatic bundled
  plugin loading for PHP and XML to Prettier executables under this checkout.
  PHP and XML fixture tests now pass with the current bundled packages.
- Lua and Ruby fixture tests still fail as no-op formatting. Current bundled Lua
  and Ruby plugin versions are not viable Prettier 3 targets and need separate
  support decisions before being advertised.
- Split formatting tests into a blocking known-passing lane and a quarantined
  Lua/Ruby lane, so CI no longer hides regressions in already-passing fixtures
  behind expected plugin-language failures.
- CI now runs an explicit `vim --version` capability check for `+job` and
  `+channel` in the blocking Vim jobs, plus a blocking `git diff --check` job.
- Added targeted config resolver tests for `g:prettier#config#plugins` and made
  the smoke lane run config resolver tests, not only the original Vim 9
  regression.
- Added project-local Prettier resolver tests proving buffer-tree lookup wins
  over Vim cwd lookup and bundled PHP plugin injection is skipped for
  project-local Prettier outside this checkout.
- `yarn lint` now uses pinned local Python requirements and a checkout-local
  virtualenv wrapper for `vim-vint`, and CI runs it as a blocking job.
- Hardened the current POSIX shell-string command path by returning raw
  executable paths from the resolver and shellescaping executable and
  `--stdin-filepath` arguments at shell call sites. Windows and argv-list job
  APIs remain open.
- Audited bundled parser-plugin viability for PHP, Ruby, XML, Lua, and Svelte.
  Docs now scope measured bundled fallback support to PHP/XML, classify Lua/Ruby
  bundled formatting as quarantined failing, and classify Svelte as detected but
  unmeasured under the current Prettier 3 baseline.
- Added a core-only formatting fixture lane for CSS, GraphQL, HTML, JS, JSON,
  Less, Markdown, SCSS, TS, Vue, and YAML so core formatter coverage is not mixed
  with bundled external plugin languages.
- The test harness can set `g:prettier#exec_cmd_path` from
  `PRETTIER_EXEC_CMD_PATH`, allowing core fixtures to run against caller-provided
  Prettier executables outside this checkout without changing package deps.
- `yarn test:formatting:core` passes against the bundled/current Prettier 3.0.3
  baseline, and `PRETTIER_EXEC_CMD_PATH=... yarn test:formatting:core:exec`
  passes against a temp Prettier 2.8.8 install outside the repo.
- Normalized the Markdown core fixture away from version-specific list
  indentation and validated the core lane against temp Prettier 2.8.8 and
  `prettier@latest` installs outside the repo.
- Added a blocking CI core-formatting matrix for caller-provided Prettier 2.8.8
  and latest executables outside this checkout.
- Added explicit bundled PHP/XML plugin injection tests and matching
  project-local PHP/XML guardrail tests proving bundled plugin flags are skipped
  outside this checkout.
- Reworked command construction to build argv-list commands for Vim/Neovim job
  APIs while preserving the existing shell-string fallback for sync/legacy paths.
- Hardened async formatting by tracking jobs per buffer, comparing
  `b:changedtick` before replacement, resetting job state on exit paths, and
  keeping manual `:PrettierAsync` from writing to disk while preserving
  autoformat-on-save writes.
- Added async safety tests for manual no-write behavior, concurrent buffer jobs,
  stale output, ignored output, and quickfix parser errors.
- Added a blocking editor smoke matrix for Vim 8.2 latest patch, stable Vim,
  Neovim 0.9 latest patch, and stable Neovim.
- Added workflow concurrency and per-job timeouts so superseded branch pushes are
  cancelled automatically and hung discovery/test lanes do not consume runners
  indefinitely.
- Changed compatibility CI to manual-only `workflow_dispatch` so compatibility
  matrices do not consume organization runners on every branch push.
- Deprecated the legacy root Dockerfile and documented Node.js 20.x plus Yarn
  Classic 1.x as the measured package-manager path.
- Added a pinned local Docker editor-matrix runner in
  `docker/editor-matrix.Dockerfile` plus `scripts/docker-editor-matrix.sh`, so
  Vim 8.2, stable Vim, Neovim 0.9, and stable Neovim smoke checks can be run
  locally without reviving the deprecated root Dockerfile.
- Added Neovim support to the checkout-local Jest/Vim harness through Neovim
  `sockconnect()` newline-mode transport, and made the editor matrix pass
  `VIM_EXECUTABLE_ARGS=--headless` for Neovim. The local Docker editor matrix
  passed for Vim 8.2, Vim 9.1 stable tag, Neovim 0.9.5, and Neovim stable.

## Security and Tooling Follow-Up

- `yarn audit --json` currently reports many path findings, but the major local
  risk is old test tooling rather than vim-prettier runtime formatting code.
- The most important vulnerable paths are pulled by `jest@23.6.0`, especially
  `jest -> jest-cli -> jest-environment-jsdom -> jsdom` and Istanbul/Babel
  coverage packages.
- Representative advisories from the current audit include:
  `babel-traverse@6.26.0` critical arbitrary code execution,
  `form-data@2.3.3` critical/high issues, `ws@5.2.3` high DoS issues,
  `request@2.88.2` moderate SSRF, and several old glob/template dependencies.
- Treat the runtime user risk as lower than the maintainer/tooling risk because
  these are dev/test dependencies, but do not ignore them. They affect anyone
  running the Jest harness against untrusted fixtures or code.
- Do not paper over these with broad Yarn `resolutions` unless a targeted
  override is proven safe. The preferred fix is to upgrade or replace
  `jest@23.6.0` and regenerate `yarn.lock`, then rerun the compatibility lanes.
- Keep compatibility CI manual-only while doing this work. Use local verification
  first, then push with `[skip ci]` unless a maintainer explicitly requests a
  manual workflow run.
- Upgraded the test harness from `jest@23.6.0` to pinned `jest@29.7.0` and
  regenerated `yarn.lock`. This removed the old jsdom/request/Babel 6/ws stack
  from the lockfile.
- Added `scripts/jest.js` so Jest 29 runs under Node 25 by deleting Node's
  experimental webstorage globals before Jest copies global descriptors. This is
  a test-harness compatibility shim, not vim-prettier runtime behavior.
- Added `jest.config.js` to keep the existing snapshot string escaping behavior,
  avoiding broad fixture snapshot churn from the Jest upgrade.
- After the Jest upgrade, `yarn audit --json` still reports unique findings for
  `brace-expansion`, `minimatch`, and `js-yaml` through Jest/Istanbul/glob
  paths, plus `nanoid` through `vim-driver -> shortid -> nanoid`.
- Replaced the `vim-driver` package dependency with a checkout-local Jest/Vim
  harness under `tests/vim-driver/`, adapted from the small subset used by this
  suite. The local harness uses deterministic process-local counters instead of
  `shortid`, removing the `vim-driver -> shortid -> nanoid` audit path without
  changing vim-prettier runtime code.
- Hardened the local Jest/Vim harness with newline-frame buffering, request
  rejection on socket close/error, connection timeout cleanup, improved child
  process cleanup, and upstream MIT license attribution.
- After removing `vim-driver`, `yarn audit --json` reported only Jest/Istanbul
  tooling paths for `brace-expansion`, `minimatch`, and `js-yaml`.
- Added targeted Yarn Classic resolutions for `brace-expansion@1.1.15`,
  `minimatch@3.1.5`, and `js-yaml@4.2.0` after a Jest 30 probe showed broader
  test-runner churn did not remove the Istanbul `js-yaml` path. The resolutions
  clear `yarn audit --json`; Yarn warns that `js-yaml@4.2.0` is outside
  `@istanbuljs/load-nyc-config`'s requested `^3.13.1` range, so keep smoke/core
  formatting verification around this override.
- Local verification used Vim 9.1 plus the Docker editor matrix for Vim 8.2,
  Vim 9.1 stable tag, Neovim 0.9.5, and Neovim stable. The manual GitHub
  Actions editor matrix remains useful as an independent runner check.

## Review Findings After b538e29

- At `b538e29`, CI full formatting discovery was non-blocking, so regressions
  in already-passing languages could be hidden.
- The blocking smoke test only protects the Vim 9 `execute()`/config-version
  regression. It does not protect actual formatting, PHP/XML plugin loading,
  async behavior, or project-local Prettier behavior.
- Project-local Prettier behavior is not fully proven because executable
  resolution still uses `getcwd()`, not the buffer's file tree.
- PHP/XML passing is scoped to bundled fallback tooling from this checkout and
  should not be treated as a project-local guarantee.
- Lua/Ruby no-op fixture failures are due to no plugin wiring in their
  ftplugins. Separate direct checks indicate the bundled plugin versions are not
  viable Prettier 3 targets.
- Svelte remains an unmeasured plugin-language gap despite being documented and
  bundled.
- Lint/tooling reproducibility is still incomplete because Node/Yarn
  expectations are not pinned or documented, and CI still lacks an
  editor/Prettier matrix beyond the current Ubuntu Vim lanes.

## Work Stages

### Stage 0: Restore a Runnable Baseline

- [x] Fix Vim 9 `redir` failure in Prettier version detection.
- [x] Add a focused regression test for version detection inside vim-driver
  `execute()`.
- [x] Run the full formatting suite and classify remaining failures by cause.
- [x] Record whether failures are harness, Vim compatibility, Prettier core, or
  external parser-plugin failures.

### Stage 1: Define Reproducible Tooling

- [x] Add an initial CI smoke workflow for the known Vim 9 regression and
  non-blocking full-suite discovery.
- [x] Split Jest formatting tests into blocking known-passing and quarantined
  known-failing lanes.
- [x] Harden CI with explicit Vim version/feature checks and `git diff --check`.
- [x] Add CI with an editor compatibility matrix after smoke lanes are stable.
- [x] Add CI with a Prettier core compatibility matrix.
- [x] Resolve `vint` reproducibility through a pinned container or installable
  local deps.
- [x] Replace or deprecate the current Dockerfile path.
- [x] Document supported Node and Yarn/package-manager versions.

### Stage 2: Prettier and Plugin Compatibility

- [x] Update classified Prettier 3.0.3 core snapshots for GraphQL, SCSS, Vue,
  and YAML without changing plugin-language snapshots broadly.
- [x] Add explicit plugin argument support for PHP/XML bundled fallback tests.
- [x] Add targeted plugin config tests for `g:prettier#config#plugins`, including
  string, list, empty, invalid, and paths with spaces.
- [x] Add project-local Prettier tests proving bundled plugin injection does not
  override local Prettier/plugins.
- [x] Validate core Prettier language fixtures on bundled/current Prettier 3.0.3
  and project-local Prettier 2.8.8.
- [x] Decide the Prettier latest core snapshot strategy for the Markdown list
  indentation delta before adding latest as a blocking target.
- [x] Audit bundled parser-plugin versions for PHP, Ruby, XML, Lua, and Svelte.
- [x] Decide Lua/Ruby/Svelte support policy together before advertising or
  removing any plugin-language support.
- [x] Decide whether bundled plugin support remains in scope.
- [x] If bundled plugins remain supported, add explicit plugin resolution without
  breaking project-local Prettier/plugin behavior.
- [x] Add or update fixtures for each supported plugin language.

### Stage 3: Command and Resolver Hardening

- [x] Fix resolver to search for Prettier from the buffer's file tree, not only
  `getcwd()`.
- [x] Avoid mutating buffer filetype defaults when merging overrides.
- [x] Harden POSIX shell-string command construction for executable paths and
  `--stdin-filepath` paths containing spaces and quotes.
- [x] Finish command construction hardening for Windows-supported job APIs.
- [x] Prefer argv-list job APIs where Vim/Neovim support allows.
- [x] Expand config-file discovery for modern Prettier config names.

### Stage 4: Async Safety

- [x] Track async jobs per buffer instead of with one global running flag.
- [x] Reset job state on every exit path.
- [x] Capture and compare `b:changedtick` before replacing async output.
- [x] Ensure manual `:PrettierAsync` does not unexpectedly write to disk.
- [x] Add tests for buffer switching, stale output, ignored files, and quickfix
  behavior.

### Stage 5: Docs and Release Prep

- [x] Sync README and `doc/prettier.txt`.
- [x] Document supported Vim, Neovim, Node, Prettier, and parser-plugin versions.
- [x] Document bundled fallback vs project-local Prettier behavior.
- [x] Add migration notes for unsupported legacy combinations.
- [x] Cut a prerelease only after CI is green for the declared matrix.

### Stage 6: Security and Test Tooling

- [x] Upgrade or replace `jest@23.6.0` to remove the stale jsdom/request/Babel 6
  dependency stack.
- [x] Prefer the smallest test-runner change that preserves vim-driver behavior,
  snapshot semantics, and manual compatibility lanes.
- [x] Run `yarn install --frozen-lockfile` after updating `yarn.lock` from a
  controlled dependency change.
- [x] Run `yarn audit --json` and record the remaining unique advisories by
  runtime vs dev/test dependency path.
- [x] Verify at minimum `yarn test:smoke`, `yarn test:formatting:core`, targeted
  async safety tests, `yarn lint`, and `git diff --check` locally before pushing.
- [x] If the Jest upgrade changes snapshot formatting or timeout behavior,
  classify those as harness changes separately from Prettier/runtime changes.
- [x] Decide whether to keep Jest 29 with targeted overrides for remaining
  `glob`/`minimatch`/`brace-expansion`/`js-yaml` dev-tooling advisories, or move
  to a newer test runner/Jest major in a follow-up slice.
- [x] Decide whether to replace `vim-driver` or its `shortid` dependency path to
  remove the remaining `nanoid` advisory.
- [x] Follow up on the remaining Jest/Istanbul `brace-expansion`, `minimatch`,
  and `js-yaml` advisory paths separately. Do not add broad `resolutions` until
  each override is proven compatible with Jest 29 and the snapshot harness.

## Verification Commands

- `yarn install --frozen-lockfile`
- `yarn test`
- `yarn test:formatting:core`
- `PRETTIER_EXEC_CMD_PATH=/path/to/prettier yarn test:formatting:core:exec`
- `yarn lint`
- `git diff --check`

If a command is not runnable, document the exact failure and whether it is a
tooling gap or a product regression.

## Review Checklist

- Does the change improve measured compatibility, or only update assumptions?
- Does it preserve project-local Prettier and config resolution?
- Are known-passing formatting languages protected by blocking CI?
- Do project-local Prettier/plugin tests prove bundled fallback is not overriding
  local tooling?
- Does CI run explicit Vim version/feature checks and `git diff --check`?
- Does it affect async buffer replacement or writes?
- Does it change support policy, install behavior, or user-facing commands?
- Are README and help docs updated when behavior changes?
- Are failures classified instead of hidden by broad snapshot updates?

---
> Source: [prettier/vim-prettier](https://github.com/prettier/vim-prettier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
