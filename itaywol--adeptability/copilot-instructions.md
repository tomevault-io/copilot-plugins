## adeptability

> <!-- adeptability:begin id=adept-code-style hash=df432118 -->

<!-- adeptability:begin id=adept-code-style hash=df432118 -->
## Go code style and conventions for the adept codebase — formatting, linters, error wrapping with sentinels, the composition-root/no-globals rule, and core invariants. Apply when writing or reviewing Go in this repo. (matches: **/*.go)


# adept code style

Conventions for Go in `github.com/itaywol/adeptability`. CI enforces formatting and lint;
the rest is reviewed.

## Formatting & lint (enforced)

- `gofmt` + `goimports` — tabs for indentation, grouped imports (stdlib / third-party /
  local). CI fails on any unformatted file.
- `golangci-lint run` (`.golangci.yml`): `errcheck` (incl. type assertions), `staticcheck`,
  `govet`, `gocritic`, `revive` (**every exported symbol needs a doc comment**), `errorlint`,
  `nilerr`, `bodyclose`, `prealloc`, `unconvert`, `misspell`, `unused`, `ineffassign`.

## Errors

- Always wrap with context and `%w`: `fmt.Errorf("clone %s: %w", url, err)`.
- Compare with `errors.Is` against the sentinels in `pkg/adept/errors.go`
  (`ErrSkillNotFound`, `ErrMergeConflict`, `ErrBudgetOverflow`, …). **Never** match on error
  strings. Need a new category? Add a sentinel there.
- `errcheck` is strict — handle or explicitly `_ =` an ignored error, and say why if it isn't
  obvious. Don't drop an error that loses data.

## Structure

- **Composition root, no globals.** Concrete implementations are wired behind interfaces into
  `*Deps` in `internal/cli/deps.go`. No package-level mutable state, no `init()` side effects.
  Take dependencies as parameters so code is testable with fakes/mocks.
- **`pkg/adept` is types-only.** Keep it dependency-light and stable; behavior lives in
  `internal/`. (Example: `SkillIDPattern` is a string in `pkg/adept`, compiled in
  `internal/canonical` — don't add a `regexp` import to `pkg/adept`.)
- Doc comments are full sentences starting with the symbol name (`// NewRoot builds …`).

## Readability

- Small, single-purpose functions; prefer **early returns** over deep nesting.
- Name things for what they are; avoid one-letter names outside tight loops/receivers.
- No magic numbers/strings — name on-disk paths and limits as constants (see
  `pkg/adept/constants.go`).
- Comments explain **why**, not what. Keep them in sync with the code — a stale comment is a
  bug.

## Invariants you must not break

1. Identity is `(id, content-hash)` — no version numbers as a sync signal.
2. Secrets never written to disk — API keys come from the environment at call time.
3. Harness models differ (per-skill / single-file / aggregator) — renderers and importers
   must respect each; aggregators parse their own section markers and honor byte budgets.
4. Canonical layout `<root>/skills/<id>/SKILL.md`; the directory name is the authoritative id.
<!-- adeptability:end id=adept-code-style -->

<!-- adeptability:begin id=adept-contributing hash=dfff4849 -->
## How to contribute to the adeptability (adept) Go CLI — build, the required pre-PR gates, conventional commits, and where things live. Apply when changing adept's own source, opening a PR, or adding a harness.


# Contributing to adept

You are working on the `adept` CLI itself (Go 1.25, module
`github.com/itaywol/adeptability`). Read [AGENTS.md](../../AGENTS.md) for architecture and
invariants; this skill is the operational checklist.

## Build & run

```bash
go build ./...
go build -o /tmp/adept ./cmd/adept && /tmp/adept --help
```

## Gates you MUST pass before opening a PR (same as CI)

```bash
gofmt -l .            # must print nothing → fix with: gofmt -w .
go vet ./...
golangci-lint run     # config: .golangci.yml
go test -race ./...
go test -run E2E ./cmd/adept   # end-to-end against a freshly built binary
```

If any gate fails, fix it before pushing — CI runs the identical set on
ubuntu/macos/windows.

## Where things live

| You're changing… | Go there |
| --- | --- |
| public types / interfaces / sentinel errors | `pkg/adept/` (types only — no behavior) |
| canonical field shape | `pkg/adept/` struct tags **and** `pkg/adeptschema/*.schema.json` together |
| a command | `internal/cli/commands_*.go`; wire deps in `internal/cli/deps.go` |
| a harness renderer | `internal/render/<id>/` + golden fixtures in its `testdata/` |
| sync / drift logic | `internal/harness/orchestrator.go` |
| 3-way merge | `internal/merge/` |
| safety scanner | `internal/scan/` |

## Conventional commits (release-please reads these)

`feat:` → minor · `fix:`/`perf:` → patch · `feat!:`/`BREAKING CHANGE:` → major ·
`refactor:`/`docs:`/`chore:`/`test:`/`ci:` → no release. The squash-merge title becomes the
changelog entry, so write it well.

## Conventions

- Wrap errors with context: `fmt.Errorf("doing X: %w", err)`. Match with `errors.Is` against
  the sentinels in `pkg/adept/errors.go` — never on error strings. Add new sentinels there.
- No package-level state, no `init()` side effects: dependencies flow through `*Deps`.
- New behavior ships with tests; bug fixes ship with a regression test. Update golden
  fixtures in the same PR when output changes, and say why in the message.
- Don't hand-edit `CHANGELOG.md`, `.release-please-manifest.json`, or version strings —
  release-please owns them.
<!-- adeptability:end id=adept-contributing -->

<!-- adeptability:begin id=adept-writing-tests hash=3b89aaf4 -->
## How to write tests in the adept Go codebase — table-driven tests with testify, golden fixtures under testdata/, the cmd/adept e2e harness, temp-dir/HOME isolation, and coverage gates. Apply when adding or changing Go tests here. (matches: **/*_test.go)


# Writing tests for adept

## Defaults

- **Table-driven** tests with `github.com/stretchr/testify/require`. One `t.Run(tc.name, …)`
  per case; name cases for the behavior they pin.
- Put unit tests in the same package (`package foo`) for white-box access; use
  `package foo_test` when you want to assert only the public surface.
- Use `t.TempDir()` for any filesystem work — never write into the repo or a shared path.
- No network in unit tests. Use `net/http/httptest` for HTTP clients, and the package's
  interfaces + fakes for `git`, the registry, and the LLM provider (see how `Deps` is wired).

## Golden fixtures

Renderers are pinned by golden files in each `internal/render/<id>/testdata/`. When you
change rendered output **on purpose**:

1. Update the fixture in the same PR.
2. Explain why in the commit message.

Never loosen an assertion to make a diff pass — fix the code or update the golden deliberately.

## End-to-end tests

`cmd/adept/*_test.go` build the real binary and drive commands against temp dirs with an
isolated environment:

```go
env := []string{
    "PATH=" + os.Getenv("PATH"),
    "HOME=" + t.TempDir(),          // isolate user config
    "ADEPT_LIBRARY=" + libRoot,     // isolate the library
}
```

Guard the slow ones so `go test -short` skips them:

```go
if testing.Short() { t.Skip("skipping e2e under -short") }
```

Assert on **observable behavior** — exit codes, files on disk, stdout/JSON — not internals.
Exit codes: `0` clean · `1` error · `2` drift/dirty or merge conflict.

## Run them

```bash
go test ./...                  # fast
go test -race ./...            # CI gate
go test -run E2E ./cmd/adept   # e2e only
go test -cover ./...           # per-package coverage
```

## Coverage gates

Keep ≥80% on `internal/render`, `internal/status`, `internal/budget`, `internal/locks`, and
`internal/canonical`. Prefer tests that would actually catch a regression (error paths, edge
cases, round-trips) over coverage-padding happy-path calls.
<!-- adeptability:end id=adept-writing-tests -->

<!-- adeptability:begin id=agents hash=d2119851 -->
## Imported from AGENTS.md


# AGENTS.md

Guidance for AI coding agents working on the **adeptability** (`adept`) codebase.
Humans: see [README.md](README.md) for usage and [CONTRIBUTING.md](CONTRIBUTING.md) for the contributor workflow.

## Project Overview

`adept` is a single-binary Go CLI for **cross-harness AI skill portability**: you author a
skill once in a canonical format and `adept` renders it accurately into every AI coding
harness in your project — Claude Code, Cursor, Codex, GitHub Copilot, OpenCode, and any
config-driven adapter you register — then keeps the two sides in sync in both directions.

- **Language:** Go 1.25, [Cobra](https://github.com/spf13/cobra) command surface.
- **Module:** `github.com/itaywol/adeptability`. Binary: `adept` (entrypoint `cmd/adept`).
- **No runtime services.** Everything is local filesystem + `git` + optional network
  (GitHub API, skills.sh, an LLM provider for the optional intent pass).
- **Source of truth is the filesystem.** Content hashes — not version numbers — drive every
  sync decision. There is no central database.

## Commands

User-facing CLI (five verbs + three subcommand groups):

| Command | Description |
| --- | --- |
| `adept init [--from <url>] [--ref <branch>] [--name <local>] [--mode symlink\|copy]` | Scaffold `.adeptability/`, optionally clone a library, adopt existing harness files |
| `adept status` | Project state at a glance: init, libraries, harnesses, drift |
| `adept sync [--harness <id>] [--force] [--dry-run]` | Push canonical skills → every enabled harness |
| `adept sync-from [--harness <id>] [--all] [--force] [--dry-run]` | Adopt harness-side edits back into canonical |
| `adept diff [--harness <id>]` | Show drift between canonical and rendered output |
| `adept harness {add\|remove\|list}` | Manage enabled harnesses |
| `adept skill {add\|install\|update\|info\|search\|check\|edit\|remove\|list}` | Manage canonical skills (local + skills.sh/GitHub) |
| `adept library {add\|remove\|list}` | Manage remote skill-library remotes |
| `adept config {list\|get\|set\|unset\|llm ...}` | Strict-typed project config |

Global flags: `--json`, `--log-level debug|info|warn|error`, `--project <path>`, `--library <path>`.
Run `adept <cmd> --help` for the authoritative, always-up-to-date surface.

## Architecture

```
cmd/adept/            main(): injects build info, calls cli.NewRoot, maps errors→exit codes
internal/cli/         Cobra composition root. One file per command group. NO package state.
pkg/adept/            STABLE public types + interfaces + sentinel errors (no behavior here)
pkg/adeptschema/      embedded JSON Schemas (skill / adapter / org / config) for validation
internal/canonical/   parse skill.yaml & SKILL.md frontmatter → *adept.Skill, schema-validate
internal/render/<h>/  one package per built-in harness: Renderer + Import (reverse render)
internal/adapter/     config-driven (YAML) harness adapters: load, validate, synthesize
internal/harness/     orchestrator: sync / sync-from / drift detection across all harnesses
internal/merge/       3-way merge + diff3 for sync-from conflict handling
internal/library/     centralized + multi-library skill resolution (first-wins on collision)
internal/scan/        static safety scanner (+ optional LLM intent pass)
internal/registry/    github (trees API) + skillssh (skills.sh catalog) clients
internal/git/         git clone/pull/checkout-at-SHA wrapper
internal/{fsutil,locks,hash,config,project,log,budget,org}/  supporting primitives
```

### Core invariants — do not break these

1. **`pkg/adept` holds types, not behavior.** It must stay dependency-light (import-free
   where possible — e.g. `SkillIDPattern` is a string, compiled in `internal/canonical`).
   In-process consumers (tests, future LSP/plugins) depend on it; keep it stable.
2. **Composition root, no globals.** `cli.NewRoot` wires every concrete implementation
   behind an interface into a `*Deps` container. No package-level state, no `init()` side
   effects. Every command takes its dependencies explicitly so it can be unit-tested with
   mocks. Add a new dependency by extending `Deps`, not by reaching for a singleton.
3. **Identity is `(id, content-hash)`.** Skills carry no version field; the hash is the
   answer to "did this change". Do not introduce version numbers as a sync signal.
4. **Canonical layout:** a skill is a directory `<root>/skills/<id>/` with one `SKILL.md`
   (YAML frontmatter + markdown body) plus optional sidecars (`scripts/`, `references/`,
   `assets/`). The directory name is the authoritative id. Skill ids use the
   harness-compatible charset `^[a-z0-9](?:[a-z0-9-]{0,48}[a-z0-9])?$` (no underscore).
   Per-skill, per-harness overrides live in an optional `harness:` map (keyed by harness id)
   plus a promoted `model` field; renderers merge their entry last via `common.MergeOverride`,
   and the schema forbids overriding identity fields. Currently consumed by claude-code and cursor.
5. **Harness models differ — renderers must respect them:** per-skill (Claude, OpenCode),
   single-file (Cursor — drops sidecars), and aggregator (Codex/Copilot — concatenate into
   one file with section markers under a byte budget). Aggregators must parse their own
   markers on `Import` and degrade to a single synthesized skill when markers are absent.
6. **Secrets never touch disk.** `config.json` records *which* LLM provider/model is used;
   API keys are resolved from the environment (`ANTHROPIC_API_KEY`) at call time only.

### Exit codes (see `cli.ExitFromError`)

- `0` clean · `1` generic error · `2` dirty/drift (`ErrDirty`) or merge conflict (`ErrMergeConflict`).
- Safety scan worst-severity maps to the same scheme: `clean`/`low`/`medium` → 0, `high` → 1, `critical` → 2.

## Key Integration Points

- **`pkg/adept`** — `HarnessAdapter`, `Renderer`, `Skill`, `RenderOutput`, `DriftReport`,
  `ImportedSkill`, sentinel errors (`ErrSkillNotFound`, `ErrMergeConflict`, …), on-disk
  layout constants (`BaseDirName`, `SkillsDirName`, …). Start here to understand contracts.
- **`pkg/adeptschema/*.schema.json`** — embedded JSON Schemas. Changing a canonical field
  means updating the schema *and* the Go struct tags in `pkg/adept` together.
- **`internal/cli/deps.go`** — the `Deps` wiring. New commands are constructed from `Deps`.
- **`testdata/` golden fixtures** under each `internal/render/<h>/` package pin exact output.

## Development

```bash
go build ./...                 # build everything
go build -o /tmp/adept ./cmd/adept   # build the binary
go test ./...                  # fast tests
go test -race ./...            # race detector (CI gate)
go test -run E2E ./cmd/adept   # end-to-end (builds the binary, drives real commands)
go vet ./...
gofmt -l .                     # must print nothing
golangci-lint run              # config in .golangci.yml
```

### Dogfooding

This repo is itself an adept skill library — committed skills live in `skills/`, and the
project-canonical layout (`.adeptability/`, `.claude/`) is **regenerated on demand and
gitignored**. To regenerate and verify rendering locally:

```bash
adept init --from "$(git rev-parse --show-toplevel)" --name adept --mode copy
adept harness add claude-code
adept sync
adept status
```

## Code Style

- **Formatting:** `gofmt` + `goimports`. CI fails on any unformatted file — run `gofmt -w .`
  before committing. There is no separate formatter to learn.
- **Linting:** `.golangci.yml` enables `errcheck`, `staticcheck`, `govet`, `gocritic`,
  `revive` (exported symbols need doc comments), `errorlint`, `nilerr`, `bodyclose`,
  `prealloc`, `unconvert`, `misspell`, and more. Run `golangci-lint run` locally.
- **Errors:** wrap with context — `fmt.Errorf("doing X: %w", err)`. Compare with
  `errors.Is` against the sentinels in `pkg/adept/errors.go`; add a new sentinel there
  rather than matching on error strings. Never silently drop an error that loses data.
- **Naming & shape:** small, single-responsibility functions; prefer early returns over
  deep nesting; doc comments on every exported symbol (full sentences, starting with the
  symbol name).
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/). Release-please
  derives the next semver from the types (`feat` → minor, `fix`/`perf` → patch,
  `feat!`/`BREAKING CHANGE:` → major; `refactor`/`docs`/`chore`/`test`/`ci` → no bump).

## Testing

- **Table-driven tests** are the default. Use `testify/require` for assertions.
- **Golden fixtures** live in `testdata/` beside each renderer; they pin exact rendered
  bytes. When you intentionally change output, update the fixture in the same commit and
  explain why in the message.
- **E2E** (`cmd/adept/*_test.go`) builds the real binary and drives commands against temp
  dirs with an isolated `HOME` and `ADEPT_LIBRARY`. Guard slow paths with
  `if testing.Short()`.
- **Coverage gates:** keep `internal/render`, `internal/status`, `internal/budget`, and
  `internal/canonical` at ≥80%. Tests should catch regressions, not pad coverage.

## Adding a harness

**Built-in (Go) adapter** — for harnesses needing custom logic:

1. Implement `adept.HarnessAdapter` in `internal/render/<id>/`.
2. Add golden fixtures under that package's `testdata/`.
3. Register it in `internal/cli/deps.go` (`registerBuiltinAdapters`).
4. Document it in the README harness table.

**Config-driven adapter** — for harnesses expressible declaratively (no code, no rebuild):

Drop a `<id>.yaml` adapter in `~/.adeptability/adapters/` matching
`pkg/adeptschema/adapter.schema.json` (`kind` = `per-skill` | `aggregator-single` |
`aggregator-per-glob`, plus `output`, `frontmatter`, `body`, `detect`, `import` hints).

## Publishing

- **Versioning:** `release-please` opens/maintains a release PR from Conventional Commits;
  merging it tags `vX.Y.Z`.
- **Release:** the tag triggers `goreleaser` (cross-compiled archives for darwin/linux/windows
  × amd64/arm64), `checksums.txt`, cosign signing, and build-provenance attestation via
  `actions/attest-build-provenance` (immutable-release safe). Docker publish to GHCR is opt-in
  via the `DOCKER_PUBLISH` repo variable.
- **Distribution (live):** GitHub release tarballs, `go install`, the `scripts/install.sh`
  curl installer, Homebrew tap (`itaywol/homebrew-tap`), and GHCR images.
- **Distribution (not wired yet):** Scoop, WinGet, and the npm wrapper (`scripts/npm/`,
  `@itaywol/adeptability`) are unpublished — tracked in the "additional package managers"
  issue, gated on a thumbs-up before we commit to maintaining them.
- Do not hand-edit `CHANGELOG.md`, `.release-please-manifest.json`, or version strings;
  release-please owns them.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **adeptability** (3310 symbols, 11238 relationships, 248 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> Index stale? Run `node .gitnexus/run.cjs analyze` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? `npx gitnexus analyze` (npm 11 crash → `npm i -g gitnexus`; #1939).

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows. For regression review, compare against the default branch: `detect_changes({scope: "compare", base_ref: "main"})`.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit changes without running `detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/adeptability/context` | Codebase overview, check index freshness |
| `gitnexus://repo/adeptability/clusters` | All functional areas |
| `gitnexus://repo/adeptability/processes` | All execution flows |
| `gitnexus://repo/adeptability/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
<!-- adeptability:end id=agents -->

<!-- adeptability:begin id=using-adept hash=c8364203 -->
## Use the `adept` CLI to author AI skills once and render them into Claude Code, Cursor, Codex, Copilot, and OpenCode. Apply when installing/syncing skills, editing skill.yaml/SKILL.md, or touching .adeptability/.


# Using the `adept` CLI

`adept` makes one AI skill portable across every coding harness. You **author once** in a
canonical format; `adept` **renders accurately** into each harness's native layout and keeps
both sides in sync.

## Mental model

```
canonical skill  ──render──▶  .claude/skills/…   (per-skill)
(.adeptability/  ──render──▶  .cursor/rules/….mdc (single-file)
 skills/<id>/    ──render──▶  AGENTS.md            (Codex aggregate)
 SKILL.md)       ──render──▶  .github/instructions (Copilot aggregate)
                 ◀─sync-from─  (adopt edits made directly in a harness file)
```

- **Identity is `(id, content-hash)`** — there are no version numbers. The hash decides
  whether something changed.
- A skill is a directory `<root>/skills/<id>/` with one `SKILL.md` (YAML frontmatter +
  markdown body) and optional sidecars (`scripts/`, `references/`, `assets/`).
- The **filesystem is the source of truth.** `config.json` only records which harnesses are
  enabled, the materialization mode, and library remotes.

## Canonical SKILL.md format

```markdown
---
id: pr-review                 # ^[a-z0-9](?:[a-z0-9-]{0,48}[a-z0-9])?$ — matches the directory name
description: Use before opening a PR. Tests, security, performance.   # <= 280 chars
activation: agent             # always | globs | agent | manual
globs: []                     # REQUIRED when activation: globs
allowed-tools: [Read, Grep]   # carried into Claude Code
targets: []                   # empty = all enabled harnesses
tags: [review, quality]
---
# PR Review Checklist
- [ ] Tests added or updated
- [ ] No secrets in the diff
```

## The commands you'll use most

```bash
adept init                       # scaffold .adeptability/ in the current project
adept init --from <git-url>      # …and clone a remote skill library to pull skills from
adept harness add claude-code    # enable a harness (claude-code | cursor | codex | copilot | opencode)
adept harness list
adept sync                       # render canonical skills → every enabled harness
adept status                     # init state, libraries, harnesses, and drift at a glance
adept diff                       # show exactly what differs between canonical and rendered
adept sync-from                  # adopt edits made directly in a harness file back to canonical
```

Authoring and sharing skills:

```bash
adept skill add my-skill --edit          # scaffold a new skill and open $EDITOR
adept skill list                         # skills resolved for this project (canonical + libraries)
adept skill search <query>               # find installable skills on skills.sh
adept skill install <owner>/<repo>/<skill>   # install one skill from GitHub/skills.sh (pinned to a SHA)
adept skill check <target>               # static safety scan (project | library:<name>:<id> | owner/repo/skill)
adept library add <name> --from <git-url>    # stack a remote library; first-wins on id collisions
```

Global flags on every command: `--json`, `--log-level debug|info|warn|error`,
`--project <path>`, `--library <path>`.

## Typical tasks

**Start fresh, author your own skills**
```bash
adept init
adept skill add lint-style --edit
adept harness add claude-code
adept harness add cursor
adept sync
```

**Adopt a project that already has harness files** (`.claude/`, `.cursor/`, `AGENTS.md`, …)
```bash
adept init        # auto-detects and adopts existing harness skills into canonical
adept status      # confirm what got adopted
adept diff        # confirm the round-trip is clean
```

**Pull skills from a shared library**
```bash
adept init --from git@github.com:my-org/skills.git
adept harness add claude-code
adept sync
```

**Install one vetted skill from the ecosystem**
```bash
adept skill search find-skills
adept skill info  vercel-labs/skills/find-skills    # repo, stars, license, SHA, installs
adept skill install vercel-labs/skills/find-skills  # preview + safety scan + y/N
```

## Rules of thumb (so you don't surprise the user)

- **Edit canonical, then `adept sync`.** Don't hand-edit rendered files like
  `.cursor/rules/*.mdc` — they're regenerated. If you must edit a harness file directly, run
  `adept sync-from` to pull the change back into canonical.
- **Run `adept status` / `adept diff` before and after** a sync so you can report what changed.
- **Installs are gated.** `adept skill install` runs a safety scan and shows a preview; a
  `critical` finding blocks the install unless the user passes `--allow-unsafe`. Never pass
  `--allow-unsafe` or `--yes` on the user's behalf without explicit confirmation.
- **Exit codes are meaningful:** `0` clean, `1` error, `2` drift/dirty or merge conflict.
  Scan severities map the same way (`high` → 1, `critical` → 2).
- **Secrets stay in the environment.** Provider API keys are read from the environment at
  call time; adept never writes them to `config.json`.
- Aggregator harnesses (Codex/Copilot) have a **byte budget** — if skills overflow it,
  the lowest-priority ones are dropped and a truncation note is written. Check `adept status`.
<!-- adeptability:end id=using-adept -->

---
> Source: [itaywol/adeptability](https://github.com/itaywol/adeptability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
