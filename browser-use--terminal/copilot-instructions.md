## terminal

> This branch is Rust-first. The old Textual app and `src/llm_browser` runtime are intentionally gone.

# Agent Notes

## Rust Rewrite Verification Loop

This branch is Rust-first. The old Textual app and `src/llm_browser` runtime are intentionally gone.

Use these checks after runtime, state, provider, worker, or TUI changes:

```bash
cargo fmt --check
cargo test
uv run --with pytest python -m pytest -q
```

Use deterministic Ratatui dumps for visual iteration:

```bash
cargo run -q -p browser-use-tui -- --state-dir /tmp/but-rust-empty --dump-screen
cargo run -q -p browser-use-tui -- --state-dir /tmp/but-rust-ready --seed-demo done --select-latest --dump-screen
cargo run -q -p browser-use-tui -- --state-dir /tmp/but-rust-running --seed-demo running --select-latest --dump-screen
cargo run -q -p browser-use-tui -- --state-dir /tmp/but-rust-browser --seed-demo done --select-latest --overlay browser --dump-screen
```

Keep any saved dump outputs under `/tmp/but-design-loop/`.

## Terminal UI Testing Standard

For any change touching `crates/browser-use-tui`, Ratatui rendering, keyboard handling, overlays, terminal output, scrollback, paste, resize behavior, completed output, or terminal state, do not claim the work is complete from `cargo test`, `cargo run`, or Ratatui/TestBackend dumps alone.

Terminal UI testing means acting like a keyboard user:

- launch the app in a real terminal, PTY, or tmux session
- send real keys for the changed workflow
- capture the visible terminal after meaningful actions
- assert expected UI text appears
- assert broken terminal artifacts do not appear: duplicate app chrome, leaked escape sequences, stale intermediate redraws, bracketed paste markers, or ANSI escapes in plain output
- inspect deterministic dumps for the relevant states

For any terminal UI change, run the required full verification command before calling the work done:

```bash
scripts/verify-terminal-ui.sh
```

This command is the repo-owned definition of done for TUI work. It runs formatting, Rust tests, Python tests, deterministic Ratatui dumps, and the real tmux terminal smoke test.

For focused real-terminal iteration, run the tmux smoke harness directly. This catches issues that dumps cannot see, including duplicated interactive panels, unconsumed arrow keys, broken bracketed paste, stale redraws, and completed output that is no longer selectable plain text:

```bash
scripts/tui-terminal-smoke.py
```

For manual terminal behavior, run the Rust TUI in a PTY:

```bash
uv run but --seed-demo done
```

Useful keys to verify: `tab`, `f2`, `ctrl+e`, `ctrl+c`, `ctrl+q`, `enter`, `esc`, paste, and arrow keys inside overlays.

After running the verification command, inspect `/tmp/but-design-loop/`, especially deterministic state dumps relevant to the change, `tui-terminal-smoke-*.txt`, and completed plain-output captures.

Before calling the TUI polished, inspect setup, ready, running, result, browser overlay, history overlay, actions overlay, developer overlay, and stopped-task states. The old visible help overlay is intentionally gone.

Final responses for TUI work must report whether `scripts/verify-terminal-ui.sh` passed, the artifact directory inspected, the specific terminal flows exercised, and any skipped step with the exact blocker.

See `docs/terminal-ui-testing.md` for the testing heuristic behind this standard.

---
> Source: [browser-use/terminal](https://github.com/browser-use/terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
