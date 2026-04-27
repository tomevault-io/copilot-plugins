## pls

> This document describes the `pls` codebase for agentic coding agents working in this repository.

# Agent Guide

This document describes the `pls` codebase for agentic coding agents working in this repository.

## Build, Test, and Run

```sh
# Build
zig build

# Run all tests
zig build test

# Run a single test by name filter
zig build test -- --filter "isDestructive detects kill"

# Run tests in one file directly (faster, no full build graph)
zig test src/agent.zig
zig test src/config.zig
zig test src/tools/shell.zig
zig test src/llm/provider.zig
zig test src/llm/json_helpers.zig

# Run the binary
zig build run -- 'your task here'
zig build run -- --dry-run 'your task here'

# Format all source files (authoritative)
zig fmt src/
```

The project uses **Zig 0.15.2**. There is no separate linter; `zig fmt` is the only formatter.

---

## Project Structure

```
src/
  main.zig            CLI entry point, arg parsing, runTask()
  agent.zig           Agent loop, tool dispatch, system prompt building
  config.zig          Config struct, TOML parser, env var overrides
  config_editor.zig   Interactive config editor (terminal UI)
  init.zig            First-run setup wizard
  tty.zig             TTY helpers (raw mode, echo control)
  llm/
    provider.zig      Shared types: Message, ToolCall, ContentBlock, Tool
    anthropic.zig     Anthropic Claude API client
    openai.zig        OpenAI API client
    gemini.zig        Google Gemini API client
    ollama.zig        Ollama local API client
    http_client.zig   Thin std.http.Client POST wrapper
    json_helpers.zig  Per-provider JSON serialisers and tests
  tools/
    shell.zig         Shell command execution, ShellResult
    confirm.zig       ask() y/N prompt, askUser() numbered options
```

---

## Architecture

### Agent Loop (`src/agent.zig`)

Turn-based loop capped at `max_turns` (default 20):

1. Call LLM with full message history
2. Print LLM text blocks as thinking — dim grey `~ text`
3. Handle `ask_user` tool calls immediately (clarification before acting)
4. Collect all `run_shell` tool calls into a batch
5. Display every command in the batch (`  $ cmd`)
6. Ask `Execute? [y/N]` once for the whole batch (respects `confirm_mode`)
7. If declined: stop immediately, return
8. Execute each command in order, print output to stderr
9. Feed all results back to LLM and repeat

### Tools

| Tool | Purpose | Dispatch |
|---|---|---|
| `run_shell` | Execute shell commands | Batched — confirmed once per LLM response |
| `ask_user` | Clarify ambiguous user intent | Immediate — answered before shell batch |

The old `confirm` tool no longer exists. Execution confirmation is fully agent-driven via `confirm_mode`.

### Confirm Modes

- `all` — prompt before every batch
- `destructive` — prompt only if any command in the batch matches `isDestructive()`
- `none` — execute without prompting

### System Prompt

Built dynamically at `Agent.init()` by `buildSystemPrompt()`. Appends a live `Environment:` section (OS, kernel, arch, hostname, user, shell, cwd, home) to the static `SYSTEM_PROMPT_BASE` string.

### Message Format

All providers share `provider.Message` internally:
- `role`: `user | assistant | tool`
- `content`: slice of `ContentBlock` — `.text` or `.tool_call`
- `tool_call_id`: set on tool-result messages to link back to the call

Wire-format differences:
- **Anthropic**: tool calls → `tool_use` blocks; results → `tool_result` in user messages
- **OpenAI/Ollama**: tool calls → `tool_calls` array on assistant message; results → `tool` role messages
- **Gemini**: tool calls → `functionCall` parts; results → `functionResponse` parts in user messages; uses function name as tool call ID; system prompt goes in `system_instruction`

---

## Code Style

### Imports

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;           // always alias this
const builtin = @import("builtin");            // only when compile-time OS/arch needed
const provider = @import("provider.zig");      // relative paths for internal modules
```

### Naming

| Kind | Convention | Example |
|---|---|---|
| Types, structs, enums | `PascalCase` | `ConfirmMode`, `ShellResult` |
| Functions | `camelCase` | `buildRequestBody`, `isDestructive` |
| Variables, fields | `snake_case` | `confirm_mode`, `api_key` |
| File-level constants | `SCREAMING_SNAKE_CASE` | `SYSTEM_PROMPT_BASE`, `API_URL` |
| Test names | Sentence string | `"isDestructive detects kill commands"` |

### Memory Management

- Every allocating function takes `allocator: Allocator` as its first parameter.
- Callers own returned slices and must free them.
- Use `defer allocator.free(x)` immediately after allocation.
- Use `errdefer buf.deinit(allocator)` on `ArrayList` buffers before `toOwnedSlice`.
- Structs that own heap memory expose `pub fn deinit(self: *Self, allocator: Allocator) void`.

Typical string-building pattern:
```zig
var buf: std.ArrayList(u8) = .empty;
errdefer buf.deinit(allocator);
const w = buf.writer(allocator);
try w.print("...", .{});
return buf.toOwnedSlice(allocator);
```

### Error Handling

- Return errors with `!ReturnType`; let errors propagate with `try`.
- Use named errors (`error.JsonParseError`, `error.ApiError`, `error.HttpError`).
- Catch only to remap to a typed error or to handle locally:
  ```zig
  const parsed = std.json.parseFromSlice(std.json.Value, allocator, body, .{}) catch {
      return error.JsonParseError;
  };
  ```
- Do not swallow errors silently.

### Types

- `[]const u8` for string parameters that are not mutated.
- `?T` for optional values; unwrap with `if (opt) |val| { ... }`.
- Tagged unions (`union(enum)`) for sum types — see `ContentBlock`.
- Avoid `anytype` except for generic writer parameters.

### Formatting

- 4-space indentation, no tabs.
- Opening braces on the same line.
- Run `zig fmt src/` before committing.

### Tests

- Tests live at the bottom of the file they test, after a divider:
  ```zig
  // ──────────────────────────────────────────────────────────────────
  // Tests
  // ──────────────────────────────────────────────────────────────────
  ```
- Use `std.testing.allocator`; always `defer allocator.free(result)`.
- Test names describe behaviour, not implementation: `"parseTOML skips comments"`.

---

## Release Process

Releases are fully automated via GitHub Actions (`.github/workflows/ci.yml`).

### Cutting a Release

```sh
git checkout main
git merge develop
git tag v1.2.3
git push origin main --tags
```

That's it. CI takes over from there.

### What CI Does Automatically

| Job | Trigger | Output |
|-----|---------|--------|
| `test` | push to `main` / `develop`, PRs | Build + unit tests |
| `release` | `v*` tag push | 4 platform binaries + 2 `.deb` packages |
| `publish` | after `release` | GitHub Release with all files attached |
| `update-homebrew` | after `publish` | `Formula/pls.rb` pushed to `colus001/homebrew-tap` |

### Release Artifacts

Each release produces:

```
pls-linux-x86_64        # Linux x86_64 (static musl)
pls-linux-aarch64       # Linux ARM64 (static musl)
pls-macos-x86_64        # macOS Intel
pls-macos-aarch64       # macOS Apple Silicon
pls_<version>_amd64.deb # Debian/Ubuntu x86_64
pls_<version>_arm64.deb # Debian/Ubuntu ARM64
```

### Required Secret

`HOMEBREW_TAP_TOKEN` must be set in `colus001/pls` repo secrets (Settings → Secrets → Actions).
It is a GitHub Fine-grained PAT with **Contents: Read and Write** access to `colus001/homebrew-tap`.
Without it, the `update-homebrew` job will fail but the GitHub Release will still be created.

### Version Bump Checklist

1. Update `VERSION` in `src/main.zig`
2. Update `version` in `build.zig.zon`
3. Commit, merge to `main`, tag, push

---

## Adding a New Tool

1. Create `src/tools/your_tool.zig` with execution logic.
2. Add to `TOOLS` in `src/agent.zig`:
   ```zig
   .{
       .name = "your_tool",
       .description = "What it does.",
       .properties = &[_]provider.ToolProperty{
           .{ .name = "param", .type = "string", .description = "..." },
       },
       .required = &[_][]const u8{"param"},
   },
   ```
3. Add `fn executeYourTool(self: *Agent, arguments_json: []const u8) ![]const u8` on `Agent`.
4. Dispatch it in the agent loop — immediately (like `ask_user`) or batched (like `run_shell`).
5. Update `SYSTEM_PROMPT_BASE` to describe the tool to the LLM.

## Adding a New LLM Provider

1. Create `src/llm/your_provider.zig`.
2. Implement:
   ```zig
   pub fn chat(
       allocator: Allocator,
       api_key: []const u8,
       model: []const u8,
       system_prompt: []const u8,
       messages: []const provider.Message,
       tools: []const provider.Tool,
   ) !provider.ChatResponse
   ```
3. Build request body manually; send via `http_client.post()`; parse into `provider.ChatResponse`. Return `error.ApiError` or `error.JsonParseError` on failure.
4. Register in `config.zig`: add variant to `Provider` enum; update `fromString`, `toString`, `getApiKey`, `getModel`, `getBaseUrl`.
5. Add a branch in `Agent.callLlm()` in `src/agent.zig`.
6. Add the provider to the setup wizard in `src/init.zig`.

---
> Source: [colus001/pls](https://github.com/colus001/pls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
