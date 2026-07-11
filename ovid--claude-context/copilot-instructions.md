## claude-context

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is (read first)

This repo **is** the installable `claude-context` Claude Code plugin, and it is **implemented and shipped**. The plugin itself is nested under `plugins/claude-context/`: its manifest (`plugins/claude-context/.claude-plugin/plugin.json`), `plugins/claude-context/SKILL.md`, `plugins/claude-context/scripts/` (collector + report engine + `lib/` helpers), and `plugins/claude-context/commands/setup.md`. `package.json`, `LICENSE`, `README.md`, the marketplace manifest (`.claude-plugin/marketplace.json`), and the `tests/*.test.mjs` suite stay at the repo root. Run the suite with `npm test` (`node --test tests/*.test.mjs`).

The design and plan under `docs/superpowers/` are **historical design records**, not a to-do list. Where they and the code disagree, **the code wins** — it has evolved past the plan's literal listings (most notably the snapshot is now session-keyed; see Conventions).

- **Design spec** — `docs/superpowers/specs/2026-07-07-claude-context-plugin-design.md`. The *what* and *why* (numbered §1–§10).
- **Implementation plan** — `docs/superpowers/plans/2026-07-07-claude-context.md`. The original 12-task TDD build.

`README.md` describes the shipped plugin.

## The one non-negotiable contract: instrument, not advisor

The plugin's entire purpose is that **agents and subagents can query their own resource state** (context %, 5h/7d usage, cost, compactions) and act on the *user's* rules — e.g. checkpoint before Claude Code's silent auto-compaction drops CLAUDE.md guidance, or pause before a rate limit. For that to work, the plugin must **only report facts**:

- **No** recommendations, evaluative/level/traffic-light labels, policy ("should compact", 70/85/90 thresholds), or derived/predictive metrics. Raw measurements and provenance only.
- The plugin supplies facts; the **user's instructions** (CLAUDE.md, skills, subagent prompts) supply any policy. A tool with baked-in thresholds would fight the ones the user wrote.
- This is enforced by a denylist test over **`report.mjs`'s rendered output** (plan Task 9) — **not** by grepping `plugins/claude-context/SKILL.md`, which legitimately uses words like "makes no recommendations" to *describe* the contract. Don't reintroduce a doc-vocabulary grep; it false-positives on the contract's own wording.

## Architecture

Two cooperating pieces plus the skill that ties them together:

- **Collector `plugins/claude-context/scripts/collect.mjs`** — an *opt-in* statusline script. Claude Code pipes stdin JSON to it (~300ms cadence); it writes a private per-session snapshot (`snapshot-<session-id>.json`, atomic, `0600`, throttled) keyed off stdin's `session_id`, and prints one compact line. Exposes `buildSnapshot`, `compactLine`, and `runCli()`.
- **Report engine `plugins/claude-context/scripts/report.mjs`** — what the skill runs. Resolves to exactly three states, keyed on whether a collector snapshot has ever been written: **collector** (a fresh snapshot exists → full numbers, `source: collector`), **collector idle** (a snapshot exists but is stale → context stays accurate — from the transcript against the snapshot's real window size, `source: transcript`, or from the stale snapshot's own `context` field, `source: collector` — while 5h/7d/cost go `null`, flagged `unavailable_stale`), and **not set up** (no snapshot has ever been written → zero transcript I/O, everything `null` including `compactions`, `source: unavailable`, `rate_limits_status: unavailable_collector`, `setup_required: true`). Emits a human block **and** a machine-readable JSON block.
- **Shared `plugins/claude-context/scripts/lib/*`** — dependency-free single-responsibility helpers (`config-dir`, `token-math`, `time`, `transcript`, `snapshot`). There are four I/O entry points — `collect.mjs`, `report.mjs`, `coexist.mjs` (the invisible-tap wrapper a co-existed statusline runs through), and `wiring.mjs` (the tested `settings.json`/sidecar surgery setup and teardown call); `lib/*` stays pure.

**The core constraint that explains the whole design:** Claude Code pushes `context_window`, `rate_limits`, and `cost` **only to the statusline process via stdin**. A skill/command invoked by an agent gets **no stdin** — it can read only the transcript JSONL, which has per-message token `usage` but **no `rate_limits`**. Hence: the collector exists to capture stdin into a snapshot the report can later read, and the transcript fallback (used only in the collector-idle state) is context-only.

**Provenance is first-class.** Every reported number carries `source`, `rate_limits_status` (`available` / `unavailable_plan` / `unavailable_provider` / `unavailable_stale` / `unavailable_collector`), `setup_required` (always present; `true` only in the not-set-up state), and `session_selection` (`session_id` / `unverified-snapshot`), so a consumer's rule never fires on a stale, never-set-up, or wrong-session value. A stale-but-present snapshot is a distinct state from "collector never installed."

**Session identity.** The report has no stdin, so it learns its own session id from the skill-passed `--session-id ${CLAUDE_SESSION_ID}` (substituted by Claude Code) → the `CLAUDE_CODE_SESSION_ID` env var → neither. With an id it reads its own keyed snapshot (deterministic); without one it degrades to the newest snapshot and flags `session_selection` accordingly. This is what keeps concurrent same-account sessions from reading each other's context/compaction numbers.

## Data-source facts (verified against live capture — don't re-derive)

These were confirmed by capturing a real session; the fixtures in `tests/fixtures/` encode them and guard against drift. **Do not replace the golden fixtures with synthesized data** — a made-up fixture only re-tests an assumption.

- Statusline **stdin** carries the real `context_window.context_window_size` (e.g. `1000000`) and `model.id` **with** the `[1m]` marker; `rate_limits.*.resets_at` is a Unix epoch in **seconds** (multiply by 1000 for ms).
- The **transcript** stores the **base** model id (`claude-opus-4-8`, *no* `1m` marker), so window size cannot be inferred from it. That is why the collector-idle fallback reuses the *snapshot's* real `window_size` to interpret the transcript's usage, rather than guessing from the model id — and why the not-set-up state (no snapshot ever written) skips transcript I/O entirely instead of guessing at a window size it has no way to verify.
- Native `context_window.used_percentage == 0` means "not yet populated" (fresh session, before the first API response) → fall through to token-based calc. Rate-limit `used_percentage: 0` stays a valid value.
- Statusline **stdin** carries `session_id` and `transcript_path`; the transcript file on disk is named `<session-id>.jsonl` under the cwd-encoded `projects/` dir. The report reaches its own session id via `CLAUDE_CODE_SESSION_ID` (env, present in current builds) or the skill-passed `${CLAUDE_SESSION_ID}` substitution — this is why the snapshot can be session-keyed and the own-transcript lookup is deterministic.
- `tests/fixtures/real-stdin.json` and `tests/fixtures/real-transcript.jsonl` are redacted captures used as golden mapping tests (plan Tasks 6 & 8).

## Commands and toolchain

- **Runtime:** Node ESM (`.mjs`), **no build step**. The **shipped plugin has zero third-party dependencies** — its scripts import only `node:` built-ins — and requires Node 18+ (or Bun) to run. No bundler or transpiler.
- **Dev tooling** (never shipped — it lives in the **private** root `package.json`, which the marketplace does not install): ESLint (flat config, `eslint.config.mjs`) + Prettier (`.prettierrc`), driven through the `Makefile`. `make help` lists targets; `make all` runs lint + format + test. Run `npm install` once to fetch the devDependencies. Prettier is scoped to code — `.prettierignore` excludes markdown, `docs/`, `paad/`, `.superpowers/`, and the vendored `claude-hud/`.
- **Tests:** Node's built-in runner — `npm test` (or `make test`) runs `node --test tests/*.test.mjs`. Single file: `node --test tests/<name>.test.mjs`. The glob is scoped to `tests/` so the vendored `claude-hud/` suite is excluded.
- No network or credential access at runtime; all tests are local-only.

## Conventions

- **Snapshot path:** `<config-dir>/claude-context/snapshot-<session-id>.json`, file mode `0600` in a `0700` dir, where `<config-dir>` is `$CLAUDE_CONFIG_DIR` when set and non-empty, otherwise `~/.claude`. The collector keys the filename by stdin's `session_id`; the report resolves the matching file from its own session id (or falls back to the newest `snapshot-*.json`). Writer and reader must resolve the path — and sanitize the session id — identically (`plugins/claude-context/scripts/lib/config-dir.mjs`).
- **Setup/teardown wiring** (`plugins/claude-context/commands/setup.md`, `commands/teardown.md`) **co-exists** rather than replaces: on a foreign slot it wraps the existing statusline via `scripts/coexist.mjs` (invisible tap — original's line unchanged), storing the whole original `statusLine` object in a `0600` `coexist.json` sidecar so teardown restores it verbatim. It **never** replaces/seizes a slot it doesn't own, stays **1-deep**, and fails safe. The risky `settings.json`/sidecar surgery is a tested module (`scripts/wiring.mjs`, marker-based ownership + a single `0600` full-file backup + atomic writes), not prose. **POSIX + Git Bash only** this alpha — native PowerShell is unsupported (no settings change). The report engine is **unchanged** (co-existence yields a normal `collector` snapshot).
- **Docs commits go directly to `main`** (the repo's established convention; there is no build/CI gate on docs).
- **Tests stay at the repo root** (`tests/*.test.mjs`) even though the plugin moved under `plugins/claude-context/` — they import the nested scripts by path but are not themselves shipped as part of the plugin.

## The `claude-hud/` reference (gitignored, not shipped)

`claude-hud/` is a vendored reference checkout of an existing statusline plugin (MIT, © Jarrod Watts) — it is **reference only**, gitignored, and never a shipped component. It is the source of the borrowed `lib/` patterns and the setup command strings. When implementing, cross-check the real shapes in `claude-hud/src/{stdin,types,transcript,external-usage}.ts` and `claude-hud/commands/setup.md`. Note the key difference: claude-hud gets stdin because it *is* the statusline, so it never faces the "no stdin" problem — do not copy its assumption that the real window size is always available. Borrowed `lib/` files must carry an attribution header; claude-hud is acknowledged in the README and LICENSE.

---
> Source: [Ovid/claude-context](https://github.com/Ovid/claude-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
