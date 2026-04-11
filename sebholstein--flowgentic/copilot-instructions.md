## flowgentic

> The Gemini CLI driver **cannot support interactive permission requests**.

# Permission Request — Gemini Driver

## Status: Not Supported

The Gemini CLI driver **cannot support interactive permission requests**.

## Why

The Gemini driver launches the CLI as a subprocess (`exec.CommandContext`) with a prompt via `-p` flag. It communicates through:

- **stdout** — buffered line-by-line, collected into a single message after the process exits.
- **stderr** — connected to `os.Stderr` directly.
- **Hooks** — `AfterAgent` and `Stop` for lifecycle events.

There is **no programmatic API, SDK, JSON-RPC protocol, HTTP server, or callback mechanism** to intercept tool permission requests. The Gemini CLI handles permissions internally:

- With `--yolo` flag: all tools are auto-approved.
- Without `--yolo`: the CLI prompts the user interactively on its own stdin/tty — which is not available in headless mode.

When running headless without `--yolo`, the CLI either:
1. Blocks waiting for tty input (and eventually times out or hangs), or
2. Auto-denies tool usage.

There is no way to inject a permission response into the running Gemini process from the outside.

## What Would Need to Change

For Gemini to support interactive permissions, Google would need to add one of:
- A Go SDK with a permission callback (like Claude's `WithCanUseTool`)
- An HTTP server mode with a permission endpoint (like OpenCode's `POST /session/:id/permissions/:permissionID`)
- A JSON-RPC protocol with server-initiated requests (like Codex's `item/commandExecution/requestApproval`)
- A stdin-based protocol for programmatic permission responses

Until then, the Gemini driver must run with `--yolo` for any meaningful tool usage in headless mode.

## Recommendation

- Do **not** advertise `CapPermissionRequest` for the Gemini driver.
- Document that Gemini requires yolo mode for headless operation.
- No code changes needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebholstein)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sebholstein)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
