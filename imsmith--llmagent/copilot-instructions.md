## llmagent

> Active sprint through Feb 28, 2026. Full plan: `/home/imsmith/Documents/remote.vault.001/src/099 Katachora/SPRINT-2026-02.md`

# LLMAgent — Elixir Agent Framework

## Sprint Context

Active sprint through Feb 28, 2026. Full plan: `/home/imsmith/Documents/remote.vault.001/src/099 Katachora/SPRINT-2026-02.md`

LLMAgent is Phase 3 (Feb 23-28). Read the sprint plan for day-by-day tasks.

## What LLMAgent Is

A GenServer-based framework for building autonomous AI agents that invoke system tools in a controlled manner. The agent talks to an LLM API, parses tool-call JSON from responses, dispatches to tool modules, formats results, and loops back to the LLM.

## Current State (as of Feb 23, 2026)

**Build: 0 errors. Tests: 96 doctests + 147 tests, 0 failures.**

### What Works
- **Comn v0.4.0** — GitHub dependency, compiles clean
- **Bash tool** — tested, proper error handling
- **Web tool** — GET/POST with headers/params, tested
- **Crypto tool** — full ed25519 & ecdsa, key generation, signing/verification
- **Inotify tool** — full implementation: watch/poll/stop/list via background `inotifywait` ports, managed by Watcher GenServer
- **EventBus** — Registry-based pub/sub, tested
- **EventLog** — in-memory immutable log with query API
- **LLMAgent.Events** — public event emission with automatic context enrichment from Comn.Contexts
- **Context propagation** — per-prompt Comn.Contexts with request_id, trace_id, actor; flows through tool dispatch and into events
- **RequireBinary utility** — checks system binaries exist
- **Agent lifecycle** — tested: multi-turn tool loops, stop/restart, concurrent agents, event ordering, resilience, memory persistence
- **LLMClient behaviour** — pluggable LLM API client; OpenAI-compatible default implementation
- **Memory behaviour** — pluggable agent memory; ETS implementation via Comn.Repo.Table.ETS with per-agent isolation
- **AgentSupervisor** — DynamicSupervisor for multiple concurrent agents (start/stop/list)

### What's Still TODO
- **Manual test** — agent prompts, dispatches tool, returns result (end-to-end with real LLM)
- **git tag v0.2.0**

### Completed This Sprint
- ~~**Inotify tool** — perform/2 returns :not_implemented~~ — Fully implemented with Watcher GenServer, background ports, event buffering
- ~~**Event wiring** — emit_event exists but tools don't emit events~~ — `LLMAgent.Events.emit/4` is public, context-enriched; Inotify Watcher emits watch_started/watch_stopped/fs_event; agent emits prompt/llm_response/tool_dispatch/invocation/error events
- ~~**Context wiring**~~ — Comn.Contexts integrated: per-prompt context with request_id/trace_id, propagated through dispatch, attached to all events
- ~~**Agent lifecycle tests** — need proper coverage~~ — 19 lifecycle tests covering context propagation, multi-turn loops, stop/restart, concurrency, event ordering, resilience, memory persistence
- **Bug fix**: `dispatch_tool` now calls `Code.ensure_loaded/1` before `function_exported?/3` — tools that weren't referenced elsewhere (File, Net, etc.) would fail dispatch through the agent
- ~~**Tool output standardization**~~ — All tools now return structured/parsed data instead of raw command text. Proc info parses /proc status into maps, Systemd status uses `systemctl show` for key-value output, Systemd list parses into service maps, Udev parses lsblk JSON + lsusb/lspci into structured lists, Net ping extracts RTT, DBus list parses busctl output. Udev list error handling fixed (was silently swallowing failures).
- **Doctests** — 96 doctests across all 25 public modules
- **README** — comprehensive rewrite documenting architecture, tools, events, utilities, supervision tree
- ~~**LLMClient behaviour**~~ — `LLMAgent.LLMClient` behaviour + `LLMAgent.LLMClient.OpenAI` impl. Agent state carries `llm_client` module, defaults to OpenAI. Extracted `call_llm/3` → client `chat/2`. `handle_info` matches `{:ok, content}` string instead of `%Req.Response{}`.
- ~~**Memory behaviour**~~ — `LLMAgent.Memory` behaviour + `LLMAgent.Memory.ETS` impl via Comn.Repo.Table.ETS. Agent persists history on every append + terminate. Restores history on restart if table survives. Designed for future tuple-space backend.
- ~~**DynamicSupervisor**~~ — `LLMAgent.AgentSupervisor` wraps DynamicSupervisor. Application starts DynamicSupervisor + default agent. Runtime `start_agent/stop_agent/list_agents` API.

## Depends On

**Comn** v0.4.0 as GitHub dependency:
```elixir
{:comn, github: "imsmith/comn", tag: "v0.4.0"}
```

## Sprint Goal for LLMAgent

**Exit criteria:**
```
mix compile   -> 0 errors (with Comn dependency)
mix test      -> all pass
manual test   -> agent prompts, dispatches tool, returns result
git tag       -> v0.2.0
README        -> documents architecture, tools, usage
```

## Key Decisions This Sprint

1. ~~**Comn integration**: Add as path dependency~~ — Done. GitHub dep at v0.4.0.
2. ~~**Startup requirements**: Make wg/ssh-keygen/gpg optional~~ — Done. Warns instead of crashing.
3. ~~**Tool output format**: Standardize ALL tools to `{:ok, %{output: term(), metadata: map()}} | {:error, %ErrorStruct{}}`~~ — Done. All tools return structured/parsed data.
4. ~~**Inotify**: Implement or delete — no stubs~~ — Implemented. Watcher GenServer + watch/poll/stop/list actions.
5. ~~**Events**: All tool invocations emit events; agent emits prompt/response/error events~~ — Done. Public `LLMAgent.Events.emit/4` with context enrichment.
6. ~~**LLM client abstraction**: Extract HTTP call behind behaviour~~ — Done. `LLMAgent.LLMClient` behaviour, OpenAI default.
7. ~~**Memory abstraction**: Pluggable per-agent memory~~ — Done. `LLMAgent.Memory` behaviour, ETS default via Comn.
8. ~~**DynamicSupervisor**: Multiple concurrent agents~~ — Done. `LLMAgent.AgentSupervisor` with start/stop/list API.

## Architecture

```
prompt(agent, content)
  -> GenServer.call
    -> set Comn.Contexts (request_id, trace_id, actor, role, model)
    -> emit :prompt event
    -> append to history (+ persist to memory)
    -> async Task: state.llm_client.chat(history, opts)
      -> {:ok, content} | {:error, reason}
        -> tool_call? dispatch_tool(tool, action, args)
          -> set tool/action in context
          -> Tools.tool() -> Code.ensure_loaded -> module.perform(action, args)
          -> emit :invocation event (with context, timing)
          -> format result -> append to history (+ persist) -> loop
        -> not tool_call? append assistant message (+ persist) -> done
```

### Supervision Tree

```
LLMAgent.Supervisor (one_for_one)
├── Task.Supervisor (LLMAgent.TaskSup)
├── Inotify.Watcher
├── DynamicSupervisor (LLMAgent.AgentSupervisor)
│   ├── LLMAgent (name: LLMAgent)    ← default, started by Application
│   └── LLMAgent (name: :agent_2)    ← started at runtime via AgentSupervisor.start_agent/1
├── Registry (EventBus)
└── EventLog
```

### Event Flow

```
LLMAgent.Events.emit(type, topic, data, source)
  -> attach context (request_id, trace_id, correlation_id) from process dictionary
  -> EventStruct.new(type, topic, enriched_data, source)
  -> EventLog.record(event)
  -> EventBus.broadcast(topic, event)
```

Agent events: `agent.prompt`, `agent.llm_response`, `agent.tool_dispatch`, `agent.error`
Tool events: `tool.<name>` (invocation), `tool.inotify` (watch_started/stopped), `tool.inotify.event` (fs_event)

### Inotify Architecture

```
Inotify.perform("watch", args) -> Watcher.start_watch(path) -> Port(inotifywait -m) -> watch_id
Inotify.perform("poll", args)  -> Watcher.poll(watch_id)    -> drain buffered events
Inotify.perform("stop", args)  -> Watcher.stop_watch(id)    -> close port, return final events
Inotify.perform("list", args)  -> Watcher.list_watches()    -> active watch IDs and paths
```

Watcher is a supervised GenServer (`LLMAgent.Tools.Inotify.Watcher`) in the application supervision tree.

## Available Tools

| Tool | Module | Status | Actions |
|---|---|---|---|
| bash | Tools.Bash | Working | exec |
| web | Tools.Web | Working | get, post |
| file | Tools.File | Working | read, write, delete |
| crypto | Tools.Crypto | Working | sha256, hmac, generate_key, generate_keypair, sign, verify |
| dbus | Tools.DBus | Working | list (parsed), introspect, call |
| systemd | Tools.Systemd | Working | status (parsed props), start, stop, restart, list (parsed) |
| proc | Tools.Proc | Working | list (structured), info (parsed /proc status) |
| net | Tools.Net | Working | list_interfaces (JSON), ping (RTT extracted), resolve |
| udev | Tools.Udev | Working | list (lsblk JSON + lsusb parsed), info (parsed), usb (parsed), pci (parsed) |
| inotify | Tools.Inotify | Working | watch, poll, stop, list |

## Key Modules

| Module | Purpose |
|---|---|
| `LLMAgent` | Main GenServer — prompt handling, tool dispatch, history, pluggable client + memory |
| `LLMAgent.LLMClient` | Behaviour for LLM API clients (`chat/2`) |
| `LLMAgent.LLMClient.OpenAI` | OpenAI-compatible chat completions client |
| `LLMAgent.Memory` | Behaviour for agent memory backends (`init/store/fetch/delete/list/teardown`) |
| `LLMAgent.Memory.ETS` | ETS-backed memory via Comn.Repo.Table.ETS |
| `LLMAgent.AgentSupervisor` | DynamicSupervisor — `start_agent/stop_agent/list_agents` |
| `LLMAgent.Events` | Public event emission with context enrichment |
| `LLMAgent.EventBus` | Registry-based pub/sub |
| `LLMAgent.EventLog` | In-memory immutable event log with query API |
| `LLMAgent.Tools` | Tool registry — maps atom names to modules |
| `LLMAgent.Tools.Inotify.Watcher` | GenServer managing inotifywait ports and event buffers |
| `LLMAgent.Tool` | Behaviour: `describe/0`, `perform/2` → `tool_result()` |

## Dependencies

```elixir
req ~> 0.5.0           # HTTP client
jason ~> 1.4           # JSON
b58 ~> 1.0             # Base58 encoding
mix_test_watch ~> 1.1  # dev-only TDD
comn v0.4.0            # GitHub dep (errors, events, secrets, contexts, repo)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imsmith)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/imsmith)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
