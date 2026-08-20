## local-codex-bridge

> Local Codex Bridge is a thin Windows-oriented MCP stdio control surface for native Codex sessions. Its purpose is to let ChatGPT or another MCP client supervise official Codex threads without creating a second task system, transcript store, queue, retry loop, or authority layer.

# AI Agent Guidance

## Project intent

Local Codex Bridge is a thin Windows-oriented MCP stdio control surface for native Codex sessions. Its purpose is to let ChatGPT or another MCP client supervise official Codex threads without creating a second task system, transcript store, queue, retry loop, or authority layer.

Keep the bridge thin. Native Codex owns persistent threads, turns, history, final messages, and execution capabilities. Bridge-owned state is limited to bounded live supervision data, pending requests, terminal snapshots, optional bounded checkpoints, and the optional UX projection.

## Architecture

The runtime chain is:

```text
MCP client -> Local Codex Bridge (JSON-RPC stdio)
           -> official `codex app-server --listen stdio://` (JSONL stdio)
           -> native Codex sessions
```

`src/mcp.ts` owns the MCP boundary, `src/app-server.ts` owns the official child-process protocol, `src/tools.ts` owns the public tool contract, and `src/runtime.ts` owns ephemeral live state. `src/checkpoint.ts` provides the separate optional local checkpoint store. The Windows Tray and Secure MCP Tunnel integration are optional layers; the Tunnel itself is external to this repository.

The seven public tools have distinct semantics:

- `codex_threads`: list/search/read persistent native threads; filters are not access control.
- `codex_turn`: create or resume a native thread and start a turn; acceptance is not completion.
- `codex_observe`: read bounded live state or explicitly degraded persisted history after Bridge state loss.
- `codex_steer`: append a semantic correction to the exact active turn; do not use it as a timer or retry.
- `codex_respond`: answer one real pending app-server request using its raw ID and exact scope.
- `codex_interrupt`: interrupt one exact native turn; it is not process control.
- `codex_checkpoint`: maintain optional bounded supervisor cognition metadata; it is not a transcript or lifecycle database.

Preserve these distinctions, the tool names, validation, annotations, and stdout protocol purity unless a requested contract change explicitly requires otherwise.

## Security and permission boundaries

The Bridge is not an OS sandbox and must not claim to be one. Codex permissions come from the official runtime plus the selected sandbox and approval policy. Prompts constrain intended behavior; they do not reduce native process capability by themselves.

Treat thread visibility, the Bridge process environment, and the local OS user as trust boundaries. The app-server child inherits the Bridge environment. Do not add secrets to fixtures, logs, examples, profiles, or command lines, and do not assume transport sanitization provides hostile multi-tenant isolation.

Never fabricate approval or user-input request IDs. A response must match an actually pending request's ID, thread, method, and turn scope. Do not silently widen `cwd` filters into security claims. Avoid stdout diagnostics because stdout is reserved for MCP JSON-RPC; operational diagnostics belong on stderr and still require redaction.

Do not modify a user's Codex installation, Tunnel authentication/profile, unrelated services, or system permissions unless the user explicitly places them in scope. Live smoke tests create persistent native threads and therefore require deliberate authorization.

## Adapting to another machine

Inspect the current checkout and target machine before deciding how to install or run it. Do not copy maintainer-specific paths, profiles, ports, PIDs, readiness URLs, or executable locations.

The published implementation currently requires Node.js 24+ and claims Windows support only. New-thread working directories must be absolute Windows drive-letter paths. Resolve the official Codex executable from the target machine's `PATH` or an explicit `CODEX_EXE`; do not add an npm Codex runtime dependency.

For a strict MCP stdio client or Tunnel, execute the built entry point with Node directly. `npm start` is suitable for a terminal but npm lifecycle output can corrupt a strict stdout protocol stream. Treat Secure MCP Tunnel setup as external configuration. If the optional Tray is needed, discover and supply that machine's readiness URL, Tunnel profile, and executable, then preserve its exact-process identity checks and no-auto-restart behavior.

Prefer evidence from the current source, tests, package metadata, and actual machine over old paths or remembered deployment state. If documentation and behavior diverge, establish the source/test truth before changing claims. Do not claim macOS or Linux support without implementation changes and real validation.

## Acceptance expectations

A rebuild or install should demonstrate, as applicable:

- dependency installation, type checking, build, and the regular automated test suite succeed;
- the MCP server keeps stdout clean, initializes correctly, and exposes exactly the seven intended tools;
- the official app-server executable is resolved and launched with the expected stdio arguments;
- persistent native history and ephemeral Bridge state remain clearly separated;
- no author-specific path, credential, Tunnel profile, port, or secret has entered the repository or generic setup;
- optional Tray/Tunnel behavior is validated only when requested, against the exact target-machine identity and configuration;
- live Codex smoke testing is reported separately from deterministic tests, including its persistent-thread side effect.

Use these as outcome criteria, not as a fixed command-by-command procedure. Adapt to the machine, preserve the trust boundaries, and report observed evidence and remaining limitations precisely.

---
> Source: [zoeynine/Local-Codex-Bridge](https://github.com/zoeynine/Local-Codex-Bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
