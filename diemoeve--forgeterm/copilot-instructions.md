## forgeterm

> > Guardian Agent daemon for AI coding CLIs. Rust. Apache 2.0.

# Forgeterm

> Guardian Agent daemon for AI coding CLIs. Rust. Apache 2.0.

## What This Is

A background daemon (`forgeterm-agent`) that auto-discovers AI coding tools (Claude Code, Codex, Gemini CLI, Cursor, Aider) on a Linux machine, monitors what they do (files, network, memory, commands), enforces resource limits via cgroups, and alerts on suspicious behavior. Optional TUI client (`forgeterm`) connects to the daemon for a live dashboard.

**Not a wrapper.** Users keep using their AI tools normally. The daemon watches from the background via /proc.

## Architecture

```
forgeterm-agent (daemon, always running)
|-- Discovery:     /proc/*/cmdline scanning every 5s
|-- Producers:     FileMonitor, NetworkMonitor, ResourceMonitor, OutputMonitor, ProcessMonitor
|-- Event Bus:     tokio broadcast channel (all signals flow through)
|-- Consumers:     RuleEngine, AuditLogger, AlertSender, CgroupGovernor, IpcServer
|-- IPC:           Unix socket (JSON-RPC) at ~/.local/share/forgeterm/agent.sock
|-- Config:        ~/.config/forgeterm/agent.toml + security-rules.toml
|-- Audit:         ~/.local/share/forgeterm/audit/YYYY-MM-DD.jsonl

forgeterm (TUI client, optional)
|-- Connects to daemon via Unix socket
|-- ratatui + crossterm
|-- Views: Sessions, Resources, Security, Tasks
```

## Cargo Workspace

```
crates/
  agent/     -> forgeterm-agent binary (daemon)
  cli/       -> forgeterm binary (TUI client)
  shared/    -> forgeterm-shared library (types, protocol, config)
```

## Coding Rules

- Linux-first. macOS/Windows later.
- Platform-specific code behind traits + `#[cfg(target_os)]` modules
- Every producer implements `trait SignalProducer { async fn run(&self, tx: Sender<Signal>); }`
- Every consumer implements `trait SignalConsumer { async fn run(&self, rx: Receiver<Signal>); }`
- Adding a new signal source or consumer = one file, no changes to core
- No `unwrap()` in production code. Use `anyhow` for error handling.
- Audit log format: JSON Lines, one entry per line, never changes schema without version bump
- IPC protocol: JSON-RPC over Unix socket, defined in shared/protocol.rs
- Tests: unit tests per module, integration test that spawns daemon + verifies discovery

## Key Types (crates/shared/)

- `CliType`: ClaudeCode, Codex, GeminiCli, Cursor, Aider, Custom(String)
- `Session`: id, pid, cli_type, status, working_dir, rss_bytes, cpu_percent, memory_limit
- `SessionStatus`: Running, Idle, HighMemory, Leaking, Restarting, Exited
- `SecurityEvent`: SensitiveFileAccess, BoundaryViolation, UnexpectedNetwork, InjectionPattern, ProcessOom
- `Severity`: Info, Warning, Critical
- `AuditEntry`: timestamp, session_id, cli_type, event, severity

## Dependencies

Agent: tokio, serde, serde_json, toml, tracing, tracing-subscriber, nix, clap, regex, dirs, notify-rust, anyhow, inotify (linux)
CLI: ratatui, crossterm, tokio, serde, serde_json, clap, anyhow
Shared: serde, serde_json (minimal)

## Resource Governor

Three modes (configurable in agent.toml under `[governor]`):
- `warn` — notifications only, no cgroups
- `throttle` — set memory.high (soft limit, kernel throttles), no hard kill. DEFAULT.
- `kill` — set both memory.high and memory.max (hard OOM kill)

Uses cgroups v2: memory.high = soft/throttle, memory.max = hard/kill.

Tiered notifications:
- 85% of limit → desktop warning
- 95% of limit → urgent notification ("save your work")
- 100% (memory.high hit) → process throttled by kernel
- 120% (memory.max hit) → OOM kill, daemon logs it

Per-CLI defaults:
- ClaudeCode: high=3GB, max=4GB
- Codex: high=1.5GB, max=2GB
- GeminiCli: high=2GB, max=3GB
- Cursor: high=3GB, max=4GB
- Default: high=2GB, max=3GB

Config format:
```toml
[governor]
enabled = true
action = "throttle"       # warn | throttle | kill
warn_threshold = 0.85
urgent_threshold = 0.95

[governor.defaults]
memory_high = "2GB"
memory_max = "3GB"

[governor.cli.ClaudeCode]
memory_high = "3GB"
memory_max = "4GB"
```

## Process Detection

Match `/proc/<pid>/cmdline` against patterns in agent.toml:
- claude, claude-code, @anthropic/claude-code
- codex, openai-codex
- gemini, gemini-cli
- cursor-agent, cursor
- aider

---
> Source: [diemoeve/forgeterm](https://github.com/diemoeve/forgeterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
