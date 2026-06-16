## windbg-mcp

> A single C# (.NET 8) MCP server that gives an LLM agent complete control over a Windows VM:

# WinDbgMCP — C# MCP Server for Windows VM Control & Kernel Debugging

## What This Is

A single C# (.NET 8) MCP server that gives an LLM agent complete control over a Windows VM:
VM lifecycle, kernel debugging (DbgEng COM), guest execution (vmrun), and user-mode debugging (Frida/dbgsrv/TTD).

## Build & Run

```bash
# Build
dotnet build src/WinDbgMCP.Server/WinDbgMCP.Server.csproj

# Run (stdio mode — used by MCP clients)
dotnet run --project src/WinDbgMCP.Server/WinDbgMCP.Server.csproj

# Run tests
dotnet test src/WinDbgMCP.Tests/WinDbgMCP.Tests.csproj
```

**dotnet is at:** `C:\Program Files\dotnet\dotnet.exe`
If not in PATH, use: `"/c/Program Files/dotnet/dotnet.exe"` from bash.

## Project Structure

```
src/WinDbgMCP.Server/
├── Program.cs                    # Entry point, MCP server setup, DI
├── appsettings.json              # VM creds, KDNET config, timeouts
├── Configuration/
│   └── ServerConfig.cs           # Configuration model classes
├── State/
│   ├── SystemState.cs            # State model + enums
│   ├── StateCoordinator.cs       # Precondition gate (heart of system)
│   ├── ErrorMessages.cs          # LLM-friendly error catalog
│   └── ToolResult.cs             # Result type
├── Vmware/
│   ├── VmwareManager.cs          # vmrun wrapper (all VM operations)
│   └── ProcessResult.cs          # vmrun result types
├── KernelDebug/
│   ├── DbgEngThread.cs           # Dedicated MTA thread for COM
│   ├── DbgEngManager.cs          # Kernel debug session manager
│   ├── DebugEventCallbacks.cs    # Debug event capture
│   ├── OutputCapture.cs          # Command output capture
│   ├── Interop/
│   │   ├── Constants.cs          # Blocked commands, HRESULT values
│   │   └── NativeMethods.cs      # P/Invoke for dbgeng.dll
│   └── Models/
│       └── DebugEvent.cs         # Debug event model
├── Guest/
│   ├── GuestExecManager.cs       # Guest command execution + file transfer
│   └── Models/
│       └── GuestCommandResult.cs # Command result model
├── UserModeDebug/
│   ├── FridaManager.cs           # Frida CLI wrapper
│   ├── DbgsrvManager.cs          # Remote user-mode debugging
│   └── TtdManager.cs             # Time Travel Debugging
└── Tools/
    ├── VmTools.cs                # vm_* tools (7)
    ├── VmScreenshotTool.cs       # vm_screenshot
    ├── KernelDebugTools.cs       # kd_* tools (7)
    ├── GuestTools.cs             # guest_* tools (5)
    ├── UserModeDebugTools.cs     # umd_* tools (8)
    └── MetaTools.cs              # get_system_state (1)
```

## Tool Catalog (29 tools)

### Meta (1)
- `get_system_state` — complete system state overview. Always allowed.

### VM Tools (8)
- `vm_start` / `vm_stop` / `vm_pause` / `vm_resume`
- `vm_snapshot_restore` / `vm_snapshot_list`
- `vm_screenshot`
- `vm_set_target` — switch the active VM target at runtime

### Kernel Debug Tools (7)
- `kd_connect` — attach to kernel via KDNET/serial
- `kd_disconnect` — detach from kernel debugger (resumes target)
- `kd_break` — halt running target (Ctrl+Break)
- `kd_continue` — resume target (go)
- `kd_step` — step into/over one instruction
- `kd_execute` — run any WinDbg command (execution-control commands blocked)
- `kd_wait_for_event` — wait for breakpoint/exception with timeout

### Guest Tools (5)
- `guest_run_command` — execute command in guest with stdout/stderr capture
- `guest_transfer_to_vm` / `guest_transfer_from_vm` — file copy
- `guest_list_processes` / `guest_kill_process`

### User-Mode Debug Tools (8)
- `umd_frida_attach` — attach Frida to guest process
- `umd_frida` — inject scripts, eval JS, list processes, detach
- `umd_frida_skill` — Frida best practices, API reference, usage patterns
- `umd_dbgsrv_connect` — connect to dbgsrv in guest
- `umd_dbgsrv_execute` — attach to process, run WinDbg commands, detach
- `umd_dbgsrv_skill` — dbgsrv best practices, WinDbg command reference
- `umd_ttd` — record/stop/retrieve/list TTD traces
- `umd_ttd_query` — query TTD traces *(stub — use WinDbg Preview)*

## Architecture Reference

Key design principles:

1. **Every tool calls StateCoordinator.ValidatePreconditionsAsync() BEFORE executing**
2. **Every operation has a timeout** — no blocking calls anywhere
3. **Error messages tell the LLM exactly what to do next**
4. **Execution-control commands (g, t, p) are BLOCKED in kd_execute** — use kd_continue/kd_step
5. **BSOD is detected and handled differently from normal breakpoints**
6. **DbgEng COM thread affinity** — all COM calls marshaled to dedicated MTA thread
7. **Snapshot restore resets ALL state** — KD, Frida, dbgsrv sessions all cleaned up

## Critical Rules

- **BREAK vs RUNNING**: Debug commands require BREAK state. Guest ops require RUNNING.
- **Blocked commands**: `g`, `gh`, `gu`, `p`, `t` are blocked in kd_execute. Use `kd_continue`/`kd_step`.
- **First-chance exceptions**: Passed through to the kernel (GO_NOT_HANDLED). Only second-chance exceptions break.
- **Frida CLI**: Each eval/inject spawns a fresh frida process. State is NOT preserved between calls.
- **dbgsrv attach**: Uses noninvasive mode — full read access but no breakpoints/stepping.

## ABSOLUTE RULES

1. **Language is C# (.NET 8).** This is NOT a Python project. All code is C#.
2. **Never use pybag.** DbgEng COM is accessed via ClrDebug NuGet package.
3. **Never modify files inside `.venv/` or `site-packages/`.** There is no Python venv.
4. **All MCP tools use `[McpServerTool]` attribute** from ModelContextProtocol SDK.
5. **All tools validate preconditions via StateCoordinator** before executing.
6. **appsettings.json contains secrets** — do not commit to public repos.

## MCP Client Configuration

Add to your MCP client config (`.mcp.json` in any project directory):

```json
{
  "mcpServers": {
    "windbg-mcp": {
      "command": "dotnet",
      "args": ["run", "--project", "C:\\path\\to\\windbg-mcp\\src\\WinDbgMCP.Server\\WinDbgMCP.Server.csproj"]
    }
  }
}
```

This works from **any project directory** — the path to the .csproj is absolute, so the MCP server is shared across all your projects.

---
> Source: [memoryforensics1/windbg-mcp](https://github.com/memoryforensics1/windbg-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
