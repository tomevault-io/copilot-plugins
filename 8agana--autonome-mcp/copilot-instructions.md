## autonome-mcp

> - **Blocking I/O in Async Context:** The `AutonomyServer` methods in `src/mcp.rs` (e.g., `handle_set_heartbeat`, `handle_get_wake_context`) perform synchronous database operations using `rusqlite` while holding a `std::sync::Mutex`. Since the server is running on a Tokio runtime, these blocking operations can stall the executor thread.

# Code Review for `autonome-mcp`

## Code Quality and Rust Best Practices

- **Blocking I/O in Async Context:** The `AutonomyServer` methods in `src/mcp.rs` (e.g., `handle_set_heartbeat`, `handle_get_wake_context`) perform synchronous database operations using `rusqlite` while holding a `std::sync::Mutex`. Since the server is running on a Tokio runtime, these blocking operations can stall the executor thread.
  - **Recommendation:** Use `tokio::task::spawn_blocking` for database interactions or consider an async-friendly SQLite wrapper like `sqlx` or `deadpool-sqlite`. Alternatively, for this scale, simply wrapping the critical sections in `spawn_blocking` is sufficient.

- **Manual Shell Escaping:** In `src/tasks.rs`, the `shell_escape` function manually escapes single quotes for use in `sh -c`. While it attempts to be POSIX-safe, manual escaping is error-prone.
  - **Recommendation:** Where possible, avoid `sh -c`. For redirection, use `std::process::Stdio::from` with `File` handles opened in Rust. This avoids the shell entirely and eliminates injection risks.

- **Hardcoded Paths:** There are several hardcoded paths, specifically in `src/federation.rs` and `src/tasks.rs`:
  - `/opt/homebrew/bin/node`
  - `/opt/homebrew/lib/node_modules/...`
  - `/Users/samuelatagana/Library/Application Support/Claude/claude-code/...`
  - **Recommendation:** Use the `which` crate or `std::env::path` to locate binaries dynamically, or make these paths configurable via environment variables or a config file.

- **Unwrap Usage:** There are a few `unwrap()` calls (e.g., `interval.parse().unwrap_or(15)` in `src/state.rs`) which are generally safe due to `unwrap_or`, but `src/mcp.rs` has `args.get("agent")...unwrap_or("gemini")`. While mostly safe, explicit error handling is preferred.

## Error Handling Patterns

- **Good Use of Anyhow:** The project consistently uses `anyhow::Result` for application-level error handling, which is appropriate.
- **Swallowed Errors in Launchd:** In `src/launchd.rs`, `unload_agent` ignores errors if the agent isn't loaded. This is generally good practice for idempotency, but ensuring that *unexpected* errors aren't masked is important.
- **Task Dispatch Errors:** In `src/tasks.rs`, `dispatch_task` spawns a shell command. If `nohup` or `sh` fails immediately, it is caught. However, failures *inside* the shell script (e.g., `cmd` not found) will report a successful spawn but fail later. The status check logic handles this via exit codes, which is acceptable.

## Security Concerns

- **Shell Injection Risk:** As mentioned, `src/tasks.rs` constructs a shell command string using user-provided inputs (`prompt`, `resume`, `model`). Even with `shell_escape`, this is a high-risk pattern. If `prompt` contains malicious sequences that bypass the simple escaping logic, it could lead to arbitrary code execution.
  - **Critical Fix:** Rewrite `dispatch_task` to set up file redirection in Rust and pass arguments directly to the process, avoiding `sh -c` if possible. If `sh` is required for `nohup` behavior, ensure the command arguments are passed as positional parameters to the shell script (e.g., `sh -c 'exec "$@"' -- command arg1 arg2`) rather than interpolated into the string.

- **YOLO Mode:** The `--yolo` flags (and equivalents like `--dangerously-skip-permissions`) explicitly bypass security controls in the invoked tools. While this is a design feature, it grants the MCP server broad authority. Ensure the server itself is running with the minimum necessary system privileges.

## Performance Issues

- **Sequential File I/O in List Tasks:** `src/tasks.rs::list_tasks` calls `check_task` for every task. `check_task` performs synchronous file I/O (`fs::read_to_string` on exit paths). For `limit=50`, this means 50 sequential file reads on every list request.
  - **Recommendation:** This is unlikely to be a bottleneck for a local tool, but could be improved by checking tasks in parallel or caching the "finished" state in the database so the filesystem isn't checked for completed tasks.

- **PID Reuse:** `is_running` in `src/tasks.rs` checks if a PID exists. On Unix systems, PIDs are reused. A new process could potentially reuse the PID of a finished task, causing the system to incorrectly report it as "running".
  - **Recommendation:** This is a hard problem to solve perfectly without a supervisor process. Storing the process start time or using a more robust process handle (if kept in memory) would help, but given the persistence requirement, this PID check is a standard compromise.

## Architecture and Design Patterns

- **Single Binary/Dual Mode:** The design of using a single binary for both the CLI/Server and the `run-wake`/`run-heartbeat` agents is clean and simplifies deployment.
- **Database Schema:** The SQLite schema in `src/state.rs` is simple and effective. The migration logic (`ensure_task_schema`) is a nice touch for evolution.
- **Federation:** The pattern of invoking other CLI agents (`gemini`, `codex`) via subprocesses is a practical way to integrate existing tools without linking to them as libraries (which might not even be possible).

## Missing Functionality or Edge Cases

- **Task Output Cleanup:** Task logs (`stdout`, `stderr`, `exit`) are stored in `/tmp`. There is no apparent cleanup mechanism. Over time, or after many restarts, these files will accumulate.
  - **Recommendation:** Implement a `prune_tasks` command or tool that deletes old DB records and their corresponding `/tmp` files.
- **Log Persistence:** `/tmp` is cleared on reboot on some systems (though often not on macOS). If `/tmp` is cleared, the database will point to non-existent log files.
  - **Recommendation:** Move logs to a persistent directory within `~/.local/share/autonomy-mcp/logs` to ensure they survive reboots and match the database persistence.

## Documentation Gaps

- **Setup Instructions:** While `RunInit` exists, documentation on *prerequisites* (Node.js, Gemini CLI, Claude Code, etc.) is implicit.
- **Tool Descriptions:** The MCP tool descriptions in `src/mcp.rs` are good and provide necessary context for the LLM.

## Specific File Observations

- **src/applescript.rs:**
  - Standard `osascript` invocation. The delay logic (`delay 1`, `delay 0.5`) makes it brittle to system load but is often necessary for UI scripting.

- **src/launchd.rs:**
  - `load_agent` and `unload_agent` use `libc::getuid()` to target the correct user session, which is correct for user agents.

- **src/federation.rs:**
  - `invoke_gemini` waits for output. If the command hangs, it might block. The `timeout_secs` parameter is accepted but the comment says "Capture intended timeout (not yet enforced...)".
  - **Fix:** Implement the timeout logic using `tokio::time::timeout` wrapping the process wait (if rewritten to be async) or use a `wait_timeout` crate for synchronous `Command`.

- **src/tasks.rs:**
  - `list_tasks` returns a `Vec<TaskStatus>`. `TaskStatus` contains full paths to logs. The consumer needs to read these files to get the output. This is a good design for large outputs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/8agana)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/8agana)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
