## umm

> A CLI that answers a question in a compact, terminal-shaped way instead of

# CLAUDE.md

## What umm is

A CLI that answers a question in a compact, terminal-shaped way instead of
conversational prose. It does almost nothing itself: it hands the question plus
a skill (a big prompt) to whichever coding agent the user configured, then
renders the markdown that comes back. **The product is the skill, not the code.**

## The two markdown files are the real source of truth

- **`SKILL.md`** (repo root) — the portable "brain". Defines the entire output
  shape and behavior. It is validated prompt content: reword it deliberately,
  not casually, and re-test against real queries after changing it. Root
  placement is intentional — it makes the repo installable as a standalone
  agent skill (`npx skills add <owner>/umm`).
- **`cli.md`** — a CLI-only addendum bolted on at prompt-build time. It tells the
  agent it has no edit permissions and to redirect the user (usually: run the
  agent directly instead of through umm). It is deliberately **not** part of the
  standalone skill, so a `npx skills` install never sees it.

Both are **excluded from Prettier** (`.prettierignore`) because their exact
wording/whitespace is functional.

## Build model: the skill is embedded, not read at runtime

`scripts/embed-skill.ts` reads `SKILL.md` + `cli.md`, strips frontmatter, and
writes `src/skill.generated.ts` (exports `SKILL`, `CLI_ADDENDUM`).

- `src/skill.generated.ts` is **generated and gitignored**. A fresh clone must
  run `npm run build:skill` (or `build` / `typecheck`, which do it) before the
  CLI will run.
- Editing `SKILL.md`/`cli.md` requires a rebuild to take effect.

## Imports use `.js`, but the source is `.ts`

Relative imports are written with `.js` specifiers (`./agents.js`) even though
the files are `.ts`. This is the tsc ESM convention (tsc resolves `.js` → `.ts`
source and emits real `.js`). Consequence:

- **`node src/index.ts` does NOT work** (Node won't rewrite `.js` → `.ts`).
- Dev runs through **Bun**: `bun src/index.ts <question>` (or `npm run dev`).
- npm ships the compiled `dist-npm/`, not the source.

## Commands

- `npm run dev -- <question>` — run from source via Bun.
- `npm run build` — embed skill, `tsc -p tsconfig.build.json` → `dist-npm/` (ESM,
  ES2019, Node ≥14), then `postbuild.ts` (shebang + chmod + copy README).
- `npm run build:binaries` — Bun `--compile` cross-platform binaries into
  `dist-bin/`. **Bun is required only here.** Binaries are ~60MB (embedded
  runtime; that's the floor).
- `npm run typecheck` / `npm run format`.

Node ≥14 for the published package; local dev/typecheck wants Node ≥22.18 (or
Bun) to run the `.ts` scripts directly.

## Distribution is dual

- **npm** carries portable compiled ESM (`dist-npm/`, Node ≥14). `bin` →
  `dist-npm/index.js`.
- **Bun binaries** (`dist-bin/`) for brew/direct download, built from the same
  source.

## Releasing (Changesets → npm + GitHub Release + Homebrew)

Releases are automated; **never bump the version or publish by hand.** `main` is
protected and squash-only, which the flow is built around.

- **Per PR:** run `npx changeset` and commit the generated `.changeset/*.md`
  (bump + one-line summary → changelog + release note). Chores with no user
  impact need none.
- **`.github/workflows/release.yml`** (push to `main`) runs `changesets/action`:
  with pending changesets it opens/updates the **"release: version packages"**
  PR (bumps `package.json`, writes `CHANGELOG.md`); squash-merging that PR runs
  `npm run release` (`changeset publish`) → npm publish, then the workflow builds
  `dist-bin/` binaries, cuts a **`v<version>`** GitHub Release with them attached,
  and regenerates the formula in the tap via `scripts/render-formula.sh` (new
  version and fresh sha256s).
- **`.github/workflows/ci.yml`** runs `format:check` + `typecheck` on PRs.

### Facts the pipeline depends on (verified, so an agent needn't re-discover)

- **The tap already exists:** `hexxt-git/homebrew-tap`, a public repo whose
  `Formula/umm.rb` is a per-platform **binary** formula (four url/sha256 blocks:
  darwin/linux × arm/intel, pointing at the `v<version>` release assets;
  `install` drops the downloaded `umm-*` binary in as `umm`). Users install via
  `brew install hexxt-git/tap/umm`. `render-formula.sh` reproduces this exact
  shape — keep them in sync if either changes.
- **Tag convention is `v<version>`** (matches the brew asset URLs). We set
  `createGithubReleases: false` so changesets' scoped `@hexxt/umm@x` tag doesn't
  become the release; the `gh release create v<version>` step owns the release.
- **npm auth is Trusted Publishing (OIDC), no token.** The workflow has
  `id-token: write` and npm is upgraded to latest in-job (OIDC needs a recent
  npm); the trusted publisher is configured on npmjs (publisher: GitHub Actions,
  repo `hexxt-git/umm`, workflow `release.yml`). Provenance is automatic — do not
  re-add `NPM_TOKEN`/`NODE_AUTH_TOKEN`.
- **Only the brew push still needs a secret:** `HOMEBREW_TAP_TOKEN`, a PAT with
  contents:write on `homebrew-tap` (the default `GITHUB_TOKEN` can't write to
  another repo).
- **Repo setting required:** Settings → Actions → "Allow GitHub Actions to create
  and approve pull requests" must be on, or the Version PR can't be opened.

## Architecture (src/)

- `index.ts` — entry + arg parsing. **Rule: only `argv[0]` may be a flag**, so
  `umm what does --force do` passes through verbatim. Flags: `--config` /
  `config`, `--raw`. No `--agent` flag (agent comes only from config).
- `agents/` — the adapter layer. `index.ts` is the table **as data**; siblings
  hold the behavior (`registry.ts` = install detection, `discover.ts` +
  per-agent files = model discovery, `types.ts` = shapes). Each agent has an
  `input` mode (`"stdin"` or `"arg"`) and a web-access flag that **matters** — a
  wrong flag ⇒ silent stale answers (e.g. cursor needs `--auto-review` to run its
  web-search tool). Verified: claude, antigravity, opencode, codex, cursor.
  **Reasoning effort is always forced low** (`effortArgs` per agent: `--effort low`
  for claude/agy, `-c model_reasoning_effort=low` for codex) — umm is for quick
  lookups, not deep research, so effort is fixed, not user-configurable. Don't add
  an effort picker; new agents with an effort flag set `effortArgs` to low.
- `run.ts` — builds `SKILL + CLI_ADDENDUM + config + query`, spawns the agent,
  returns stdout. **No fallback by design**: if the agent fails, surface and exit.
- `config.ts` — JSON at `$XDG_CONFIG_HOME/umm/config.json` (`~/.config/umm/…`).
  Injected into the prompt as a `## Configuration` block (length, sources).
- `wizard.ts` — dependency-free raw-mode TTY select for `umm --config` / first run.
- `spinner.ts` — stderr-only elapsed spinner (agents buffer until done).
- `render/` — hand-rolled markdown → terminal, **no deps**, full markdown minus
  HTML. `width.ts` is display-width-aware (counts columns, ignores ANSI, handles
  wide/zero-width) — the wrapping correctness depends on it, don't naïvely use
  `.length`. Color only on a TTY; piped/redirected/`--raw` emits clean markdown.

## Conventions

- Pre-commit (Husky) runs `npm run precommit` → lint-staged → Prettier. Tooling
  is npm-based; only `build:binaries` uses Bun.
- Keep the agent adapter table honest: mark flags you haven't actually tested.
- Comment sparingly. Explain a non-obvious _why_ in one terse line; never annotate
  what the code already says. Prefer no comment over a redundant one.

---
> Source: [hexxt-git/umm](https://github.com/hexxt-git/umm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
