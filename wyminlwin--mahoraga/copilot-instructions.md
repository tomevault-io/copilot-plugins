## mahoraga

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mahoraga is a terminal-based (TUI) prompt validation and improvement tool. It uses LLM providers (Azure OpenAI, OpenAI, or Anthropic) to analyze prompts and provide quality scores, improvement suggestions, and identifies unclear parts.

## Build & Development Commands

```bash
cargo build              # Build the project
cargo build --release    # Build optimized release version
cargo run -- summon      # Run the TUI application
cargo run -- --help      # Show help
```

## Architecture

**Stack**: Rust, ratatui (TUI framework), tokio (async runtime), crossterm (terminal handling)

**Key patterns**:
- Async/await for non-blocking API calls
- State machine pattern for app screens and states
- Provider trait for LLM abstraction
- TOML configuration stored at `~/.config/mahoraga/config.toml`

**Directory structure**:
```
src/
├── main.rs           # Entry point, CLI parsing (clap)
├── app.rs            # App state machine and event loop
├── config.rs         # Config loading/saving
├── types.rs          # Shared types and enums
├── providers/
│   ├── mod.rs        # Provider trait + factory
│   ├── azure.rs      # Azure OpenAI implementation
│   ├── openai.rs     # OpenAI implementation
│   └── anthropic.rs  # Anthropic Claude implementation
└── ui/
    ├── mod.rs
    ├── theme.rs      # Color palette
    ├── main_screen.rs
    ├── settings_screen.rs
    └── widgets/
        ├── mod.rs
        ├── header.rs
        ├── prompt_input.rs
        ├── score_display.rs
        ├── feedback.rs
        └── command_menu.rs
```

**App screens**: `Main` (prompt analysis) and `Settings` (API configuration)

**In-app commands**: `/settings`, `/provider`, `/clear`, `/default`, `/exit`

## Adding New LLM Providers

1. Create new file in `src/providers/` implementing the `Provider` trait
2. Implement `async fn analyze(&self, prompt: &str) -> Result<AnalysisResult>`
3. Add to `ProviderType` enum in `src/types.rs`
4. Update factory in `src/providers/mod.rs`

## Configuration

Config file location: `~/.config/mahoraga/config.toml`

```toml
[provider]
active = "openai"  # "azure", "openai", or "anthropic"

[azure]
url = ""
api_key = ""
deployment = ""
api_version = "2024-02-15-preview"

[openai]
api_key = ""
model = "gpt-4"

[anthropic]
api_key = ""
model = "claude-sonnet-4-20250514"
```

## Type Definitions

All types in `src/types.rs`:
- `Config` - Main configuration structure
- `ProviderType` - Enum for active provider (Azure, OpenAI, Anthropic)
- `AnalysisResult` - Score, improvements, unclear parts
- `Screen`, `AppState` - UI state enums
- `Command` - Available in-app commands
- `SettingsField` - Settings form fields

## Key Dependencies

- `ratatui` - TUI rendering
- `crossterm` - Terminal input/output
- `tokio` - Async runtime
- `reqwest` - HTTP client for API calls
- `clap` - CLI argument parsing
- `serde` + `toml` - Configuration serialization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wyMinLwin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
