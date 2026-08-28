## aicmd

> `README.md` covers what the tool does, requirements, usage, configuration,

# aicmd — agent guide

`README.md` covers what the tool does, requirements, usage, configuration,
the "How it works" internals, and the `make` targets. Don't repeat those
here.

## Layout

Source lives under `internal/`; the repository root holds only `main.go`,
which sets `version` and calls `cli.Run`.

- `internal/cli` — usage text, flag parsing, prompt building, and `Run`:
  parse, generate, deliver.
- `internal/config` — `Config`, the minimal TOML subset, `aicmd config`.
- `internal/provider` — `Provider`, the registry, `ModelSpec`, `Trace`,
  `CleanCommand`, and every provider implementation.
- `internal/terminal` — TIOCSTI injection, TTY detection, clipboard.
- `internal/debug` — the `AICMD_DEBUG` logger, shared by cli and terminal.

Imports run one way: `cli → {config, provider, terminal}` and
`config → provider`. Never make `provider` import `config`; anything a
provider needs from user config is passed in as `provider.Options`. See
`docs/adr/2026-08-23-internal-package-layout.md`.

## Working rules

- Run `make vet` and `make test` before considering a change done. Both must
  be clean.
- After any change to `.go` files, rebuild the repo-root binary with
  `make build` — `./aicmd` is what the user runs interactively and must
  never be left stale.
- Keep dependencies limited to the pinned official OpenAI Go SDK and its
  transitives; see `docs/adr/2026-08-21-openai-sdk-api-providers.md`. Config
  parsing remains a hand-rolled minimal TOML subset and terminal ioctls use
  raw `syscall`, not `x/term`.

## Testing instructions

- Tests live beside the package they cover (`internal/*/[name]_test.go`);
  table-driven tests are the local idiom.
- Tests never touch the real provider CLIs or the user's real config:
  `useTempConfig` points `XDG_CONFIG_HOME` at a `t.TempDir()` (there is a
  copy in `internal/cli` and `internal/config`, since it cannot be shared
  across packages), and provider tests prepend a directory of fake CLI
  scripts to `PATH`. Use these hooks when adding tests for provider or
  config behaviour.

## Conventions

- Standard `gofmt` Go style; terse, comment-where-nonobvious. Match the
  existing tone — comments explain *why* (e.g. why the prompt goes via stdin,
  why fstat isn't enough for TTY detection), not *what*.
- Error handling: plain wrapped errors (`fmt.Errorf` with `%w` where useful),
  no panic paths; `main` prints `aicmd: <err>` and exits 1.
- CLI flags: single dash is for one-letter short forms (`-m`, `-v`, `-h`),
  double dash for long forms (`--verbose`, `--version`). Go's `flag` package
  treats `-flag` and `--flag` alike, so `checkFlagDashes` in `internal/cli/flags.go`
  enforces the convention before parsing. Every new flag must follow it.
- Config additions go through `Keys` in `internal/config/config.go` and need both
  `parseValue` (file) and `setValue` (CLI, validated) arms, plus tests.
- The `-m` flag and the `model` config key share one spec string format:
  `provider/model/effort`, validated in `internal/provider/provider.go`.

## Architecture decision records (ADRs)

Design choices with lasting effect are logged in `docs/adr/`, one dated file
per decision.

If you make a new lasting design choice, add a new ADR (paragraph or two,
decision + why + guardrails, no template). Reverse a decision with a new ADR;
never edit old ones.

## Release notes

- There is no repository-wide changelog. Each GitHub release carries its own
  curated notes, and commit history is the exhaustive engineering record.
- Before tagging, review `git log` and `git diff` since the previous tag. Write
  concise Markdown notes in a temporary file outside the repo. Describe user
  outcomes, compatibility concerns, and upgrade steps—not individual commits
  or implementation details. Omit internal refactors, tests, and
  documentation-only corrections unless they materially affect users or the
  release/install process.
- Choose the SemVer bump from user impact. Create an annotated tag whose
  message is the complete release note, for example
  `git tag -a vX.Y.Z --cleanup=verbatim -F /tmp/aicmd-vX.Y.Z-notes.md`, then
  inspect it with `git tag -n99 vX.Y.Z` before pushing. `--cleanup=verbatim`
  preserves Markdown headings that Git would otherwise treat as comments. A
  version-only annotation is invalid.
- The release workflow publishes the tag annotation verbatim with
  `--notes-from-tag`; do not use generated commit-list notes. If release notes
  need correction after publishing, edit the GitHub release directly without
  moving or recreating the tag.
- After the workflow succeeds, verify the release is neither draft nor
  prerelease, all four archives and `checksums.txt` exist, the published notes
  match the tag annotation, and the anonymous README install command installs
  a binary reporting the new version.

## Guardrails

- The generated command is never executed by `aicmd` — do not add a path
  that runs it (ADR-guarded).
- The user prompt goes to every provider via **stdin** (codex `-`, claude
  piped text, agy a stream-json envelope), never as an argv word, so it
  isn't exposed in the process list. Keep it that way.
- Treat generated output as untrusted input: it is cleaned by `cleanCommand`
  and always lands at the user's prompt for review, never in a shell.
- The release workflow's asset names and `checksums.txt` format are consumed
  by `install.sh` — keep them compatible, or version them together.
- The binary is `curl | sh`-installed onto arbitrary machines; supply-chain
  surface is why dependencies are banned.

---
> Source: [zazencodes/aicmd](https://github.com/zazencodes/aicmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
