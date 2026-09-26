## grok-remote

> `grok agent` runs the Grok agent without the interactive TUI. It's the entry point used by editor integrations, headless automation, and shared-process setups. There are four subcommands plus a handful of options that apply to all of them.

# `grok agent` reference

`grok agent` runs the Grok agent without the interactive TUI. It's the entry point used by editor integrations, headless automation, and shared-process setups. There are four subcommands plus a handful of options that apply to all of them.

Captured from `grok agent --help` against:

```
grok 0.1.212 (b7b8204a484)
```

## Top-level shape

```
grok agent [OPTIONS] [COMMAND]

Commands:
  stdio     Run the agent over stdio
  headless  Run the agent headlessly over the Grok WebSocket relay
  serve     Run the agent as a WebSocket server
  leader    Run as the shared leader process for other clients
  help      Print help
```

## Options that apply to every subcommand

| Flag | Env | What it does |
|---|---|---|
| `--reauth` (alias `----reauthenticate`) | | Run authentication before starting the agent. Useful when the cached login is stale. |
| `-m, --model <MODEL>` | | Model ID to use (e.g. `grok-build`, `grok-4-fast`). |
| `--reasoning-effort <EFFORT>` | | Reasoning effort for reasoning models. Valid: `none`, `minimal`, `low`, `medium`, `high`, `xhigh`. |
| `--always-approve` | | Auto-approve all tool executions. Equivalent of headless yolo mode. |
| `--agent-profile <PATH>` | `GROK_AGENT` | Path to an agent profile file. See "Agent profiles" below. |
| `--leader` | | Connect to a shared leader process instead of starting a new agent. Lets multiple clients share one backend. Default comes from `[cli] use_leader` in `config.toml`. |
| `--no-leader` | | Start a fresh agent even when config enables leader mode. |
| `--grok-ws-origin <URL>` | | Override the WebSocket origin used by `headless`, `leader`, and `serve`. Internal. |
| `--grok-ws-url <URL>` | | Override the WebSocket URL. Internal. |
| `--cli-chat-proxy-base-url <URL>` | | Override the CLI chat proxy base URL. (This is what grok-bench's proxy intercepts when you set `base_url` in `config.toml`.) |
| `--xai-api-base-url <URL>` | | Override the public xAI API base URL. |
| `-h, --help` | | Print help. |

## `grok agent stdio`

Run the agent over standard input/output. Each line on stdin is fed to the agent; each line on stdout is its response. Designed to be embedded by another process via a child-process pipe (editor integrations, scripting, ACP/MCP clients).

```
Usage: grok agent stdio

Options:
  -h, --help  Print help
```

No subcommand-specific options. Inherits all top-level options listed above.

**Example:**
```sh
# Pipe a prompt in and read the response out.
echo '{"role":"user","content":"hello"}' | grok agent stdio --model grok-build
```

## `grok agent headless`

Run the agent headlessly over the Grok WebSocket relay. This is the mode that backs cloud sessions and any remote-relay-based client.

```
Usage: grok agent headless [OPTIONS]

Options:
      --grok-ws-origin <GROK_WS_ORIGIN>
      --grok-ws-url <GROK_WS_URL>
  -h, --help                             Print help
```

The two `--grok-ws-*` overrides duplicate the top-level ones; useful when you want to make the override explicit at the subcommand level.

**Example:**
```sh
grok agent headless --model grok-build --reasoning-effort medium
```

## `grok agent serve`

Run the agent as a WebSocket server other clients connect to over the network. Lets you run the agent on one machine and use it from another, or share one backend between many clients.

```
Usage: grok agent serve [OPTIONS]

Options:
      --bind <BIND>
          Address for the server to listen on [default: 127.0.0.1:2419]
      --secret <SECRET>
          Secret token for client authentication (auto-generated if not provided)
          [env: GROK_AGENT_SECRET=]
      --remote <REMOTE>
          Remote agent URL for proxy mode
      --grok-ws-origin <GROK_WS_ORIGIN>
      --grok-ws-url <GROK_WS_URL>
  -h, --help
          Print help
```

Boot output prints something like:

```
Grok agent server starting...
Agent server listening on ws://127.0.0.1:2419
Clients should connect with: --remote ws://127.0.0.1:2419/ws --secret <token>
```

Authentication: clients must present the secret token. If not passed via `--secret`, one is auto-generated and printed on startup; or set `GROK_AGENT_SECRET` in env.

**Example:**
```sh
# Start the server on the local network with a known token.
GROK_AGENT_SECRET=hunter2 grok agent serve --bind 0.0.0.0:2419

# Connect another grok client to it:
grok agent stdio --remote ws://server:2419/ws --secret hunter2
```

**Proxy mode** (`--remote`): forward all traffic to another running agent server instead of running the agent locally. Useful for fan-out or for putting a closer-to-user listener in front of a slower remote.

## `grok agent leader`

Run as the shared leader process other agent clients connect to via `--leader`. The leader holds session state, MCP connections, skill caches, and the model conversation; many clients can ride the same leader and share that state. Default enabled by `[cli] use_leader = true` in `config.toml`.

```
Usage: grok agent leader [OPTIONS]

Options:
      --no-exit-on-disconnect            Keep the leader running after the last client disconnects
      --no-auto-update                   Disable periodic auto-update checks for the leader
      --grok-ws-origin <GROK_WS_ORIGIN>
      --grok-ws-url <GROK_WS_URL>
  -h, --help                             Print help
```

The leader periodically checks for updates (auto-update). When an update is available **and** the agent is idle, the leader installs the update and shuts down so the next client spawns a refreshed leader. Disable with `--no-auto-update`.

By default the leader exits when the last client disconnects. `--no-exit-on-disconnect` keeps it alive (faster reconnects, holds context warm).

**Example:**
```sh
# Run a long-lived leader in the background.
grok agent leader --no-exit-on-disconnect &

# All subsequent grok / grok agent invocations attach to it.
grok agent stdio --leader   # uses the shared leader
grok agent stdio --no-leader # forces a fresh agent anyway
```

## Related: managing leaders

The top-level `grok leader` command (not `grok agent leader`) is for inspecting and stopping leader processes:

```
grok leader list                    # list running leader processes
grok leader info <pid>              # show details for one
grok leader kill                    # stop all running leaders
grok leader profile <pid>           # manage CPU profiling for a leader
  ├── status                        # show profiling status
  ├── start [--frequency-hz N]      # start CPU profiling
  └── stop  --output <PATH>         # stop and write the profile to PATH
```

## Agent profiles (`--agent-profile`)

A "profile" is a self-contained agent definition (markdown with frontmatter, or a config.toml `[agent]` block). It overrides the default agent persona, tools, and rules. The lookup order is:

1. `--agent-profile <PATH>` CLI flag
2. `GROK_AGENT` env var (path or registered name)
3. `[agent]` block in the current `config.toml`
4. Built-in default

Errors you may see:
- `error: failed to load agent profile '<path>'`
- `error: --agent-profile path is not a file: <path>`
- `error: --agent-profile path '<path>'`

## Quick reference: typical invocations

| Goal | Invocation |
|---|---|
| One-shot prompt over stdio | `echo '{"role":"user","content":"…"}' \| grok agent stdio` |
| Headless run against the relay | `grok agent headless --model grok-build --always-approve` |
| Local WS server for editor integrations | `grok agent serve --bind 127.0.0.1:2419` |
| Long-lived shared backend | `grok agent leader --no-exit-on-disconnect &` |
| One-off using a custom persona | `grok agent stdio --agent-profile ./my-persona.md` |
| Force a fresh agent (skip leader) | `grok agent stdio --no-leader` |

## Notes

- All subcommands inherit the top-level `--always-approve`, `--model`, `--reasoning-effort`, `--agent-profile`, `--leader/--no-leader`, and `--reauth` flags.
- `--grok-ws-origin` and `--grok-ws-url` exist on the parent and on `headless`/`serve`/`leader` subcommands. They're internal overrides for the WebSocket transport; you only need them when pointing at a non-default relay.
- The `--remote` flag on `serve` puts it in proxy mode: instead of running its own agent, it forwards traffic to another `serve` instance.
- Authentication on `serve`: if `--secret` is omitted and `GROK_AGENT_SECRET` is unset, a fresh secret is generated and printed at startup. Captured on stdout, so don't lose the first few lines of the log.

---
> Source: [daniel-farina/grok-remote](https://github.com/daniel-farina/grok-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
