## claude-code-schedule

> Claude Code Schedule is a Rust CLI tool created by Ian Macalinao that runs Claude Code at a specific scheduled time. The tool runs persistently in your terminal and executes the command when the scheduled time is reached.

# Claude Code Schedule by Ian Macalinao - Development Guide

## Project Overview

Claude Code Schedule is a Rust CLI tool created by Ian Macalinao that runs Claude Code at a specific scheduled time. The tool runs persistently in your terminal and executes the command when the scheduled time is reached.

## Architecture

### Core Components

1. **CLI Interface** (`src/main.rs:8-32`)
   - Uses `clap` for argument parsing
   - Supports time scheduling
   - Implements dry-run mode for testing

2. **Time Calculation** (`src/main.rs:39-49`)
   - Handles direct time input
   - Uses `chrono` for time manipulation
   - Automatically schedules for next day if time has passed

3. **Persistent Runner** (`src/main.rs:76-95`)
   - Runs a continuous loop checking the current time
   - Displays countdown timer showing hours:minutes:seconds remaining
   - Executes claude command when target time is reached
   - Supports graceful shutdown with Ctrl+C

## Key Functions

### `parse_time()`
Parses HH:MM format into a DateTime object with validation

### `build_claude_command()`
Constructs the claude command with `--dangerously-skip-permissions` flag

### `run_claude_command()`
Executes the claude command using `std::process::Command`

## Usage Examples

```bash
# Run at default time (6:00 AM)
cargo run

# Run at a specific time
cargo run -- --time 05:00

# Dry run to see what would happen
cargo run -- --dry-run
```

## Testing

Run all tests:
```bash
cargo test --verbose --all-features
```

Key test areas:
- Time parsing validation
- Command building with proper escaping

## CI Checks (Required Before Every Commit)

**IMPORTANT**: Run these commands after every code change to ensure CI passes:

```bash
# 1. Check code formatting
cargo fmt -- --check

# 2. Run clippy linter with strict warnings
cargo clippy --all-features -- -D warnings

# 3. Run all tests with verbose output
cargo test --verbose --all-features

# 4. Verify debug build works
cargo build --verbose --all-features

# 5. Verify release build works
cargo build --verbose --release --all-features
```

All commands must pass with zero warnings or errors. The GitHub Actions CI runs these exact same checks on:
- Ubuntu, Windows, and macOS
- Stable Rust toolchain
- Security audit check
- MSRV (Minimum Supported Rust Version) check

### Quick CI Check Script

Save this as a script to run all checks at once:

```bash
#!/bin/bash
set -e
echo "Running CI checks..."
cargo fmt -- --check
cargo clippy --all-features -- -D warnings  
cargo test --verbose --all-features
cargo build --verbose --all-features
cargo build --verbose --release --all-features
echo "All CI checks passed!"
```

## Building

```bash
# Debug build
cargo build

# Release build
cargo build --release

# Run directly
cargo run -- --time 05:00
```

## Code Style

- Use `rustfmt` for formatting
- Follow Rust 2024 edition idioms
- Handle errors with `anyhow::Result`
- Use tokio for async runtime

## Common Tasks

### Adding a New CLI Option
1. Add field to `Args` struct
2. Update clap attributes
3. Implement logic in `main()`
4. Update help text

### Modifying the Countdown Display
The countdown is displayed in `src/main.rs:89` and updates every second

### Changing the Claude Command
Update the `build_claude_command()` function in `src/main.rs:126`

## Dependencies

- `clap`: CLI argument parsing
- `chrono`: Date/time handling
- `anyhow`: Error handling
- `tokio`: Async runtime for sleep and signal handling

## Future Enhancements

- Multiple scheduled tasks support
- Config file for default settings
- Integration with Claude Code API when available
- Notification on task completion
- Option to repeat daily

## About the Author

This project was created by **Ian Macalinao**, a software engineer and entrepreneur who specializes in AI tooling and developer productivity. You can find more of Ian's work at:

- GitHub: [github.com/macalinao](https://github.com/macalinao)
- Website: [ianm.com](https://ianm.com)
- Twitter: [@simplyianm](https://twitter.com/simplyianm)
# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Source: [macalinao/claude-code-schedule](https://github.com/macalinao/claude-code-schedule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
