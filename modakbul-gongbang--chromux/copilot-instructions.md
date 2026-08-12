## chromux

> chromux is a zero-dependency Node.js CLI for controlling real Google Chrome

# chromux agent guide

chromux is a zero-dependency Node.js CLI for controlling real Google Chrome
profiles through raw CDP. Keep the public command surface small and let
`chromux help` be the source of truth for current CLI syntax.

## Working Rules

- Use Node.js >= 22 and avoid adding runtime dependencies unless the tradeoff is
  explicit and necessary.
- Prefer existing raw-CDP and daemon/profile patterns in `chromux.mjs` over new
  abstraction layers.
- New browser actions should usually be expressed through `run` or `cdp` before
  adding another top-level verb.
- Compatibility aliases may remain quiet, but do not expose deprecated aliases
  in the main help surface.
- Keep agent-facing instructions under `skills/`: `skills/chromux/` for CLI
  usage and `skills/chromux-work/` for browser-work orchestration. Keep
  installation/runtime setup in `install.md`.
- Keep repo-local helper runner material under `snippets/_builtin/`.
- Command responses are an agent-facing product surface, not just data: `open`
  returns `interactive`/`next`/`hints`/`scripts`/`replay`, actions return a
  `next` verification hint, failed runs append repair hints. When changing
  response fields or workflow guidance, keep the responses, `chromux help`,
  `README.md`, and both skills telling the same story — and extend
  `benchmarks/chromux-doc-check.mjs` needles so drift fails validation.
- Durable per-host agent knowledge has two write surfaces: site notes
  (`chromux note`, `~/.chromux/skills/<host>/`) for facts, and replay scripts
  (`chromux script`, `~/.chromux/scripts/<host>/`) for proven flows. Features
  that touch host knowledge should surface through both `open` responses and
  the skills.
- Observation payload size is a first-class metric (agents pay per byte read).
  Changes to `snapshot`, `open`, or response shapes should be checked with
  `benchmarks/chromux-token-benchmark.mjs`, and the README Token Footprint
  table refreshed when numbers move materially.

## High-Risk Change Scope

- Treat OOPIF target multiplexing and CDP session routing as a standalone
  high-risk change set. Do not bundle it with multiple other browser surfaces
  unless the user explicitly approves the combined scope after a time and
  verification-cost estimate.

## Validation

Run focused checks after changes:

```bash
node chromux.mjs help
./test.sh
node benchmarks/chromux-doc-check.mjs
npm pack --dry-run
```

`npm pack --dry-run` should include only the package allowlist from
`package.json`; local planning or handoff artifacts should not be published.
`chromux-doc-check.mjs` asserts that help, README, and both skills still
document the public surface — add needles when adding features.

`./test.sh` drives real Chrome and fetches `https://example.com|org|net` and
`https://news.ycombinator.com`. In a sandbox without a usable browser or open
egress, the suite still runs fully with three adjustments:

- point `chromePath` in `$CHROMUX_HOME/config.json` at any Chromium binary
  (e.g. a Playwright-managed one);
- pass sandbox-safe flags via `CHROMUX_EXTRA_CHROME_ARGS`, typically
  `--no-sandbox --disable-dev-shm-usage --disable-gpu --no-proxy-server`
  (Chromium on Linux silently adopts `HTTPS_PROXY`, which turns blocked
  egress into confusing `chrome-error://chromewebdata/` loads);
- serve the external hosts from a local HTTPS fixture and map them with
  `--host-resolver-rules="MAP example.com 127.0.0.1, ..."` plus
  `--ignore-certificate-errors` (test-profile only).

## Verification Cadence

- Use the smallest focused probe during implementation. Reserve `./test.sh`
  for coherent milestones, batched source-changing review fixes, and the frozen
  final HEAD instead of rerunning it after every small edit.
- Run cost-bearing agent benchmarks only after the product HEAD is frozen and
  required local checks pass. Randomized tasks must use a recorded deterministic
  seed; a fixed-coordinate replay against a randomized task is diagnostic
  evidence, not a passing result.

## Release / Publish

Automatic npm publishing is **disabled** (removed in commit `77fe76e`;
`.github/workflows/npm-publish.yml` no longer exists — treat any doc that says
merging to `main` publishes as stale). `.github/workflows/ci.yml` validates
pull requests and pushes; it does not publish. The canonical install path is a
repo checkout plus `npm install -g .` (see `install.md`); the npm registry
package lags the repo and is only updated by an explicit, user-requested
release.

Before pushing a change intended for a future release, still bump
`package.json` to an unpublished version — it keeps versions meaningful and a
later manual `npm publish` one command away. Never run `npm publish` unless
the user explicitly asks for a package release.

## Pre-Publish Checklist

Before committing, pushing, tagging, or publishing a final chromux change:

- Run `git status --short` and confirm the staged files are only the intended
  repo changes.
- Bump `package.json` `version` (patch for bug fixes, minor for new features,
  major for breaking CLI changes) so versions stay meaningful and a later
  explicit release is one command away. Merging to `main` does NOT publish;
  see Release / Publish above.
- Run `node chromux.mjs help` and confirm the documented public command surface
  matches the CLI output.
- Run `./test.sh` for behavioral coverage.
- Run `npm pack --dry-run` and confirm the tarball contains only the package
  allowlist from `package.json`.
- npm releases happen only when the user explicitly asks; until then the repo
  checkout (`npm install -g .`) is the supported install.
- If behavior changed, update the matching docs and skills in the same change:
  `README.md`, `install.md`, `skills/chromux/`, and `skills/chromux-work/`.
- Before finalizing, read `install.md` and the relevant files under `skills/`
  to check for stale setup, usage, or agent-facing instructions.
- For browser focus or profile behavior changes, run one live smoke check with
  the relevant `CHROMUX_PROFILE` and environment variables instead of relying on
  docs or code inspection alone.

---
> Source: [modakbul-gongbang/chromux](https://github.com/modakbul-gongbang/chromux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
