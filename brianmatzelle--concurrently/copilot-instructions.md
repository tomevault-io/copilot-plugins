## concurrently

> Concurrent subagent workflow TUI. Each agent is a real Claude Code process (`claude -p`) running in parallel with full tool access — file editing, bash, search, everything.

# concurrently

Concurrent subagent workflow TUI. Each agent is a real Claude Code process (`claude -p`) running in parallel with full tool access — file editing, bash, search, everything.

## Build & Run

```bash
export ANTHROPIC_API_KEY=sk-ant-...
cargo build --release
cargo run
```

Requires `claude` CLI installed and on PATH.

## Architecture

```
User prompt
    │
    ▼
Orchestrator (raw Anthropic API, non-streaming)
    │  decomposes task into 2-5 subtasks
    ▼
┌────────────────────────────────┐
│  tokio::spawn × N              │
│  claude -p "task"              │
│    --output-format stream-json │
│    --append-system-prompt ...  │
│    --dangerously-skip-perms    │
│  Each agent = real Claude Code │
└────────────┬───────────────────┘
             │ mpsc AgentEvents
             ▼
         App event loop (30fps)
             │
             ▼
         ratatui TUI
```

### Source Files

- `src/main.rs` — Entry point, terminal setup, 30fps event loop, keybindings
- `src/app.rs` — App state, kernel (shared conversation history), agent lifecycle, synthesizer
- `src/agent.rs` — Spawns `claude -p` processes, parses `stream-json` stdout into AgentEvents
- `src/api.rs` — Raw Anthropic API client (streaming + non-streaming), used only by orchestrator and synthesizer
- `src/orchestrator.rs` — Task decomposition: sends user prompt to Claude, gets back JSON array of subtasks
- `src/ui.rs` — Ratatui rendering: header, agent list sidebar, detail view, status bar

### Key Concepts

- **Kernel** (`Vec<Message>`): Shared conversation history. Every agent and the orchestrator see it. When agents complete, results fold back into the kernel. Persists across rounds (press `n` for new task).
- **AgentEvent**: Enum sent over mpsc channels from agent tokio tasks to the main loop. Variants: `StatusChange`, `TextDelta`, `ToolUse`, `CostUpdate`, `Finished`.
- **stream-json parsing**: Each `claude -p` outputs newline-delimited JSON. We parse `assistant` messages (extract text deltas + tool_use blocks) and `result` messages (done/error + cost).

## Key Bindings

- **Input mode**: Type task, `Enter` to submit
- **Running mode**: `↑`/`↓` select agent, `j`/`k` scroll output
- **Done mode**: `s` synthesize, `n` new task (keeps kernel), `q` quit
- `Ctrl+C` quits from any mode

## Agent Flags

Each agent spawns with:
```
claude -p <task>
  --append-system-prompt <kernel context>
  --output-format stream-json
  --include-partial-messages
  --dangerously-skip-permissions
  --no-session-persistence
  --model sonnet
```

## Distribution

- **AUR**: `yay -S concurrently-bin` — PKGBUILD lives in `~/projects/maintaining/concurrently-bin/`
- **Debian/Ubuntu**: `cargo deb` builds a `.deb` package from Cargo.toml metadata — install with `dpkg -i`
- **Homebrew**: `brew tap brianmatzelle/tap && brew install concurrently`
- **GitHub**: `brianmatzelle/concurrently`, releases have Linux x86_64 binaries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brianmatzelle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brianmatzelle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
