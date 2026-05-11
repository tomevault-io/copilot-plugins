## claude-daily

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Important**: When making changes to interfaces, features, architecture, or workflows, review and update `assets/dataflow.png` and `README.md` accordingly.

## Build and Development Commands

```bash
# Build
cargo build
cargo build --release

# Run
cargo run -- <command>

# Test
cargo test
cargo test <test_name>         # Run specific test
cargo test -- --nocapture      # Show test output

# Install locally
cargo install --path .
```

## Project Structure

```
daily/
├── src/              # Rust CLI source code (daily binary)
├── site/             # Product website (marketing/docs) - React + Vite
│                     # Run: cd site && npm run dev
├── web/              # Dashboard UI for `daily show` - React + Vite
│                     # Run: cd web && npm run dev
│                     # Served by Rust server at runtime
├── hooks/            # Claude Code hook shell scripts (installed to ~/.claude/hooks/)
├── commands/         # Claude Code slash command templates (installed to ~/.claude/commands/)
├── scripts/          # Installation scripts (install.sh)
├── assets/           # Documentation assets (dataflow.png, screenshots)
└── .github/          # CI/CD workflows
```

**Important distinction**:
- `site/` = Public website for promotion and documentation
- `web/` = User-facing dashboard embedded in `daily show` command

## Architecture Overview

Daily is a context archive system for Claude Code that automatically records and summarizes AI-assisted work sessions using Claude Code hooks.

### Data Flow

1. **Session Start Hook** (`hooks/session_start.rs`) - Creates daily folder structure (`~/.claude/daily/YYYY-MM-DD/`)
2. **Session End Hook** (`hooks/session_end.rs`) - Spawns background summarization process
3. **Summarization** (`summarizer/engine.rs`) - Invokes Claude CLI to process transcripts and generate summaries
4. **Archive Storage** (`archive/`) - Writes session and daily summary markdown files

### Module Structure

- `cli/` - Command parsing (clap) and command implementations
  - `args.rs` - CLI argument definitions
  - `commands/` - Individual command handlers (init, view, install, summarize, etc.)
- `hooks/` - Claude Code hook handlers (session_start, session_end)
  - `input.rs` - Reads hook JSON input from stdin
- `archive/` - Archive file management
  - `manager.rs` - Directory structure and file I/O
  - `session.rs` - Session archive data structure
  - `daily.rs` - Daily summary data structure
  - `templates.rs` - Markdown templates
- `summarizer/` - AI summarization via Claude CLI
  - `engine.rs` - Invokes `claude` CLI and parses JSON responses
  - `prompts.rs` - Prompt templates for summarization
- `transcript/` - Claude Code transcript parsing (`parser.rs`)
- `jobs/` - Background job management (`manager.rs`)
- `config/` - Configuration handling via confy (`settings.rs`)

### Key Patterns

- **Background Processing**: Session end hook spawns `daily summarize --foreground` as a detached process to avoid blocking Claude Code exit
- **Claude CLI Invocation**: `SummarizerEngine` calls `claude --model <model> --print -p` with prompts via stdin, parses JSON from stdout
- **Configuration**: Uses `confy` crate, config file at `~/.config/daily/config.toml`
- **Archive Structure**: `~/.claude/daily/YYYY-MM-DD/{daily.md, session-name.md}`

### Testing

Tests use `tempfile::TempDir` for isolated file system tests. Run with `cargo test`.

---
> Source: [Oanakiaja/claude-daily](https://github.com/Oanakiaja/claude-daily) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
