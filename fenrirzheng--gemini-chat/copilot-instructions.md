## gemini-chat

> Project memory for `gemini-chat` — a Rust CLI that wraps `gemini --acp` via the [agent-client-protocol](https://docs.rs/agent-client-protocol) 0.11.x SDK.

# CLAUDE.md

Project memory for `gemini-chat` — a Rust CLI that wraps `gemini --acp` via the [agent-client-protocol](https://docs.rs/agent-client-protocol) 0.11.x SDK.

## Repo shape

- Independent git repo at `/home/fenrir/Documents/gemini-acp/`. **No remote** (`git remote -v` is empty); branch is `main`.
- The parent `$HOME` is a separate dotfiles repo (remote `bash-for-fenrir`) whose [root `.gitignore`](../../.gitignore) excludes `Documents/*`. The two repos are unrelated despite the directory nesting — don't confuse `cd` paths between them.
- No submodules. Tracked: [Cargo.toml](Cargo.toml), [Cargo.lock](Cargo.lock), [src/main.rs](src/main.rs), [gemini-acp.md](gemini-acp.md), [rust-acp.md](rust-acp.md), [TECHNICAL.md](TECHNICAL.md), this file. Gitignored: `target/`.

## Build & run

```bash
cargo build --release
./target/release/gemini-chat --help
```

Runtime requirement: an ACP-speaking Gemini agent. The SDK default `AcpAgent::google_gemini()` runs `npx -y -- @google/gemini-cli@latest --experimental-acp` (so Node.js + npx is the minimum), and pays npx's package-resolution cost on every cold start (~14 s). Passing `--model M` or `--agent-cmd "gemini --acp ..."` switches to the local [`gemini`](https://github.com/google-gemini/gemini-cli) binary on `PATH` (currently 0.41.2 via nvm), dropping cold start to ~5–7 s. See [TECHNICAL.md §6](TECHNICAL.md#6-performance-characteristics).

## The four modes

One-shot, REPL, daemon, and connect — full table + flag reference lives in [rust-acp.md §4](rust-acp.md#4-本專案實作示範). Don't duplicate it here. Wire protocol for daemon/connect is specified in [TECHNICAL.md §4](TECHNICAL.md#4-mode-by-mode-reference).

## Daemon discipline

- Default socket: `/tmp/gemini-chat-$USER.sock`. Override with `--socket PATH`.
- Stale-socket detection: on bind, daemon `exists()`-checks the path, then probes with `UnixStream::connect`. Connect-success ⇒ refuse to start (another daemon owns it). Connect-fail ⇒ treat as stale, `remove_file`, then bind. Logic lives in [`run_daemon`](src/main.rs) at the top of the function.
- Proper stop: `gemini-chat --connect --shutdown`. Clean SIGINT / SIGTERM also clean up the socket file.
- If the daemon crashes mid-session: the ACP session is gone. The next connect will fail; restart with `--daemon`. There is no session resume across daemon restarts (see [TECHNICAL.md §7](TECHNICAL.md#7-known-limitations)).

## Editing src/main.rs

Read [TECHNICAL.md §5 "Design decisions log"](TECHNICAL.md#5-design-decisions-log) before changing protocol-handling code. Two specific traps:

- **Do NOT add a global `on_receive_notification` handler** on the `Client` builder. Empirically it intercepts session-update notifications and starves `session.read_update()` / `read_to_string()`. See [TECHNICAL.md §5 decision 2](TECHNICAL.md#5-design-decisions-log).
- `acp::SessionMessage` is `#[non_exhaustive]`. The `match` in [`prompt_and_collect`](src/main.rs) keeps a `_ => {}` arm — keep it when adding new variants, don't replace with exhaustive matching.

## Tests

No `cargo test` suite yet. Smoke tests are run manually; recipes are documented in [TECHNICAL.md §4](TECHNICAL.md#4-mode-by-mode-reference) and were the basis for commit `9e96d38`. Multi-turn daemon context retention is verified by hand ("favourite colour is teal" → next turn "what colour" → "teal").

## Git

- Local-only repo; per global rule in [`~/.claude/CLAUDE.md`](../../.claude/CLAUDE.md), no `git push`, no PRs, no opening a remote without asking first.
- Commit style: small system-scoped commits matching the existing log (`feat: ...`, `init: ...`).

## Don't

- Don't `git push` or `git remote add` without explicit user instruction.
- Don't commit `Cargo.lock` churn from a stray `cargo update`. Update lockfile only when bumping a real dependency.
- Don't re-add a global `on_receive_notification` handler — see [TECHNICAL.md §5](TECHNICAL.md#5-design-decisions-log).
- Don't make `match acp::SessionMessage` exhaustive — it's `#[non_exhaustive]`, will break on the next SDK release.
- Don't duplicate user shell aliases here. The user owns those at `~/.shell.d/aliases`; this repo stays alias-agnostic so the binary is reusable from any setup.
- Don't auto-spawn a daemon from `--connect` — the race window between probe and bind is real; let the user manage daemon lifecycle explicitly.

---
> Source: [FenrirZheng/gemini-chat](https://github.com/FenrirZheng/gemini-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
