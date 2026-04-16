## tzig

> tzig is a terminal multiplexer like tmux, but simpler:

## Project Overview

tzig is a terminal multiplexer like tmux, but simpler:
- No mouse capture (scrolling works natively in your terminal)
- No session management or detach/reattach
- Just scrollback history with a lightweight overlay

Built in Zig, uses Ghostty's virtual terminal library for terminal emulation.

## Build Commands

```bash
zig build          # Build the project
zig build run      # Build and run
zig build test     # Run tests
```

The built binary is at `./zig-out/bin/tzig`.

## Zig 0.15+ API Notes

This project uses Zig 0.15+. Key API differences from older versions:

### File I/O Writers
Writers require a buffer parameter:
```zig
// Correct for Zig 0.15+
var buf: [4096]u8 = undefined;
var writer = std.fs.File.stdout().writer(&buf);
const w = &writer.interface;
try w.writeAll("hello");
try w.flush();  // Don't forget to flush!

// Wrong (old API)
const writer = std.io.getStdOut().writer();  // Does not exist
```

### Static file handles
```zig
// Correct
const stdout = std.fs.File.stdout();
const stderr = std.fs.File.stderr();

// Wrong (old API)
std.io.getStdOut()  // Does not exist
```

## Project Structure

- `src/main.zig` - Main entry point, terminal proxy implementation
- `src/cli.zig` - CLI argument parsing (--help, --version)
- `build.zig` - Build configuration
- `vendor/ghostty/` - Ghostty dependency (provides ghostty-vt module)

## Architecture

The terminal proxy works by:
1. Opening a PTY (pseudo-terminal)
2. Forking a child process running the user's shell
3. Proxying I/O between the real terminal and the PTY
4. Using ghostty-vt to parse and track terminal state
5. Providing a scrollback overlay (Ctrl+] to toggle)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nounder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
