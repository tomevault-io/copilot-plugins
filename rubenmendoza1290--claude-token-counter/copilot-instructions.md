## claude-token-counter

> This file provides guidance to Google Gemini and other AI coding assistants when working with code in this repository.

# GEMINI.md

This file provides guidance to Google Gemini and other AI coding assistants when working with code in this repository.

## Project Overview

Claude Token Counter is a Rust-based CLI tool that helps users monitor their Claude token usage through two approaches: (1) Real-time parsing of local Claude Code JSONL logs for all users, and (2) API-based tracking for Team/Enterprise users with Admin keys. The tool provides accurate cost estimation, beautiful terminal visualization, and comprehensive token tracking across all token types (input, output, cache creation, cache read).

## Current Workflow Phase

**Phase**: Core Implementation Complete
**Status**: Beta - Live monitoring fully functional, API integration complete, ready for enhancements

### Workflow Checklist

- [x] Project initialization with Cargo
- [x] Dependency configuration
- [x] Basic CLI structure with subcommands
- [x] Documentation suite (README, development guide)
- [x] Git repository setup and GitHub integration
- [x] SSH authentication configuration
- [x] Configuration module implementation
- [x] API client module implementation
- [x] Data models definition
- [x] Status command implementation
- [x] History command implementation
- [x] Config command implementation
- [x] Local JSONL parsing module
- [x] Live monitoring command
- [x] Cost calculation engine
- [x] Terminal formatting and display
- [x] Error handling and validation
- [ ] File watching (instant updates)
- [ ] Filtering by date/project/model
- [ ] Export functionality (CSV/JSON)
- [ ] Unit and integration tests
- [ ] Cross-platform testing (Linux/Windows)
- [ ] CI/CD automation
- [ ] Pre-built binary releases

## Build and Development Commands

### Building
```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release

# The binary will be at target/release/claude-token-counter
```

### Running
```bash
# Run directly with cargo
cargo run -- live                    # Live monitoring (recommended)
cargo run -- live --refresh 5        # Custom refresh interval
cargo run -- status                  # API-based current status
cargo run -- history --days 7        # API-based history
cargo run -- config --api-key KEY    # Configure API key

# Run the built binary
./target/release/claude-token-counter live
```

### Testing
```bash
# Run all tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific test
cargo test test_name
```

### Code Quality
```bash
# Format code
cargo fmt

# Check formatting without modifying
cargo fmt -- --check

# Run clippy (linter)
cargo clippy

# Run clippy with all warnings
cargo clippy -- -W clippy::all
```

### Development Workflow
```bash
# Check code compiles without building
cargo check

# Build and run in one step
cargo run -- live

# Watch mode (requires cargo-watch)
cargo watch -x "run -- live"
```

## Architecture

### High-Level Design

The application follows a modular architecture with dual data sources:

1. **CLI Layer** (`main.rs`): Command-line interface using `clap` with derive macros. Defines four subcommands (status, history, config, live) and handles argument parsing. Contains live monitor implementation with terminal control.

2. **Local Module** (`src/local/`): Parses Claude Code JSONL logs from `~/.claude/projects/`. Recursively discovers all JSONL files, deserializes log entries, extracts token usage data, and aggregates statistics. Works for all users without requiring API keys.

3. **API Client Module** (`src/api/`): Handles communication with Anthropic Usage API. Supports two endpoints for regular API usage and Claude Code specific usage. Requires Admin API keys (Team/Enterprise only).

4. **Configuration Module** (`src/config/`): Manages API key storage and user preferences. Stores config in `~/.config/claude-token-counter/config.json` using the `dirs` crate for cross-platform home directory resolution.

5. **Display Module** (`src/display/`): Formats and presents data to the terminal using `colored` for styled output. Includes status display, history visualization, progress bars, and formatted tables.

6. **Models Module** (`src/models/`): Serde-based structs for API responses including UsageRecord, UsageResponse, and UsageSummary for aggregated statistics.

### Key Design Decisions

- **Dual-Mode Architecture**: Supports both local file parsing (universal) and API integration (enterprise), making the tool valuable for all user types
- **Local-First Approach**: Prioritizes local JSONL parsing as the primary feature since it works for all users and provides more granular data
- **Async Runtime**: Uses Tokio for async operations to handle API requests and non-blocking sleep in live monitor
- **Real-Time Monitoring**: Implements live refresh using crossterm for terminal control and tokio::sleep for intervals
- **Comprehensive Token Tracking**: Tracks all token types (input, output, cache creation, cache read) for accurate cost calculation
- **Error Handling**: Uses `anyhow` for flexible error handling with context. Gracefully handles malformed JSONL lines
- **Config Storage**: Stores API keys securely in user's config directory (never in the repository)
- **CLI Framework**: Uses `clap` with derive macros for type-safe, self-documenting CLI

### Anthropic API Integration

When working with the API:
- Base URL: `https://api.anthropic.com/v1`
- Usage endpoints:
  - `/v1/organizations/usage_report/messages` - Regular API usage
  - `/v1/organizations/usage_report/claude_code` - Claude Code usage
- Requires Admin API key in `x-api-key` header
- API version header: `anthropic-version: 2023-06-01`
- Date range support with ISO 8601 format (YYYY-MM-DD)
- Only available for Team/Enterprise accounts

### Local JSONL Parsing

Claude Code stores conversation logs at `~/.claude/projects/` with structure containing message, model, usage, and timestamp fields. The local module recursively walks directories, parses JSON lines, and aggregates token usage across all files.

### Module Structure

```
src/
├── main.rs           # Entry point, CLI definition, live monitor
├── api/
│   └── mod.rs       # API client implementation
├── config/
│   └── mod.rs       # Config persistence and loading
├── display/
│   └── mod.rs       # Terminal output formatting
├── local/
│   └── mod.rs       # JSONL parsing and aggregation
└── models/
    └── mod.rs       # Data models for API responses
```

### Dependencies Rationale

- `clap 4.5`: Industry-standard CLI framework with excellent derive support
- `tokio 1.40`: De facto async runtime for Rust
- `reqwest 0.12`: High-level HTTP client built on tokio
- `serde 1.0` + `serde_json`: Standard serialization/deserialization
- `chrono 0.4`: Date/time handling for usage history and timestamps
- `colored 2.1`: Terminal color and styling
- `dirs 5.0`: Cross-platform config directory resolution
- `anyhow 1.0`: Ergonomic error handling for applications
- `notify 7.0`: File system watching (prepared for future instant updates)
- `walkdir 2.5`: Recursive directory traversal for finding JSONL files
- `crossterm 0.28`: Terminal control for live monitor screen clearing

## Key Decisions & Context

### Idea & Validation

**Core Idea**: Provide all Claude users with transparent, real-time visibility into token usage and costs, not just enterprise customers.

**Target Audience**:
- Individual developers with Claude Pro accounts
- Enterprise teams with Admin API access
- Anyone using Claude Code who wants to understand their usage

**Validation Status**: Beta release with working live monitoring feature. Successfully tested with 38M+ tokens across real projects.

### Technical Decisions

**Dual-Mode Architecture**: Implemented both local file parsing (works for everyone) and API integration (Team/Enterprise only) to maximize utility across user types.

**Local-First Strategy**: After discovering Anthropic API requires Admin keys, pivoted to prioritize local JSONL parsing as primary feature. This democratizes token tracking.

**Cost Calculation**: Implemented accurate pricing based on official Claude Sonnet 4.5 rates ($3/$15/$3.75/$0.30 per MTok for input/output/cache-write/cache-read).

**Real-Time Updates**: Used crossterm for terminal control and tokio::sleep for polling intervals. Future enhancement will add file watching for instant updates.

**Error Resilience**: Parser gracefully skips malformed JSONL lines with warnings rather than failing, ensuring tool works with imperfect data.

### Creative Strategy

**User Experience**: Focus on immediate utility - the `live` command should "just work" without configuration for most users.

**Visual Design**: Professional terminal interface with color-coded output, formatted numbers (commas), and clear hierarchical information display.

**Security Model**: Never store credentials in repository, use file system permissions, avoid credential exposure in logs or error messages.

**Accessibility**: Works for all users regardless of subscription tier or API access.

## Session History

### Session 2025-12-01
- **Phase**: Foundation & Setup
- **Accomplishments**:
  - Created new Cargo project with complete dependency configuration
  - Implemented CLI skeleton with three subcommands (status, history, config)
  - Created comprehensive documentation suite (README.md, CLAUDE.md, AGENTS.md, GEMINI.md)
  - Configured .gitignore for Rust projects
  - Initialized Git repository and pushed to GitHub
  - Set up SSH authentication with GitHub using Ed25519 key
- **Key Decisions**:
  - Chose Rust with Tokio async runtime for implementation
  - Selected Clap with derive macros for CLI parsing
  - Decided on ~/.config/ for cross-platform config storage
  - Designed modular architecture with planned separation of concerns
- **Next Steps**:
  - Implement configuration module with secure file persistence
  - Build API client for Anthropic API integration
  - Create data models for API responses
  - Implement actual functionality for each subcommand

### Session 2025-12-02
- **Phase**: Core Implementation
- **Accomplishments**:
  - Implemented complete local JSONL parsing module (src/local/mod.rs)
  - Built fully functional live monitoring command with real-time updates
  - Implemented API client with support for two Anthropic endpoints
  - Created config module with file-based persistence
  - Built display module with colored terminal output
  - Defined data models for API responses
  - Implemented accurate cost calculation for all token types
  - Added terminal control with crossterm for live screen updates
  - Tested with real data showing 38M+ tokens
- **Key Decisions**:
  - Pivoted to local-first approach after discovering API requires Admin keys
  - Implemented dual-mode architecture (local + API) for universal compatibility
  - Used walkdir for recursive JSONL discovery across all projects
  - Implemented graceful error handling that skips malformed lines
  - Chose polling with configurable refresh over immediate file watching (prepared for future enhancement)
- **Next Steps**:
  - Implement file watching with notify crate for instant updates
  - Add filtering by date range, project, and model
  - Create export functionality (CSV/JSON)
  - Build model-specific usage breakdowns
  - Add configurable usage alerts
  - Cross-platform testing on Linux and Windows
  - Set up CI/CD for automated releases

## Working Instructions

### Current Focus

The tool is now in beta with core features complete. Priority is enhancing functionality and preparing for wider release:

1. Implement file watching for instant updates (replace polling)
2. Add filtering and export capabilities
3. Create comprehensive test suite
4. Cross-platform validation
5. Set up automated builds and releases

### Development Workflow

1. **Before Coding**: Review AGENTS.md and session summary to understand current state
2. **Implementation**: Follow the module structure outlined in Architecture section
3. **Testing**: Write tests alongside implementation
4. **Documentation**: Update README and AGENTS.md as features are completed
5. **Commits**: Make atomic commits with clear messages describing what was implemented

### Code Style Guidelines

- Follow standard Rust formatting (cargo fmt)
- Address all clippy warnings
- Use descriptive variable names
- Add doc comments for public functions and modules
- Prefer explicit error handling over unwrap/expect

### Security Considerations

- API keys are stored in `~/.config/claude-token-counter/config.json`
- This file should have restricted permissions (0600)
- Never log or display full API keys
- The config file is gitignored to prevent accidental commits
- Local JSONL files may contain conversation data - handle with care

## Repository Information

- **GitHub**: https://github.com/rubenmendoza1290/claude-token-counter
- **Remote**: git@github.com:rubenmendoza1290/claude-token-counter.git
- **Branch**: master
- **License**: MIT

---
> Source: [rubenmendoza1290/claude-token-counter](https://github.com/rubenmendoza1290/claude-token-counter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
