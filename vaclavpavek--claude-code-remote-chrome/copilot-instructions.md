## claude-code-remote-chrome

> Chrome Bridge is a transparent TCP proxy for Chrome Native Messaging Host (NMH) Unix sockets. It enables Claude Code's built-in MCP server (`claude --claude-in-chrome-mcp`) to communicate with Chrome running on the host from inside a Docker container.

# CLAUDE.md — Technical context for AI agents

## Architecture

Chrome Bridge is a transparent TCP proxy for Chrome Native Messaging Host (NMH) Unix sockets. It enables Claude Code's built-in MCP server (`claude --claude-in-chrome-mcp`) to communicate with Chrome running on the host from inside a Docker container.

### Data flow

```
Claude Code (container)
  → claude --claude-in-chrome-mcp (MCP server, spawned by Claude)
    → connects to Unix socket: /tmp/claude-mcp-browser-bridge-{USER}/{PID}.sock
      → socat (entrypoint.sh, forwards to TCP)
        → bridge-host.js (TCP :9229, forwards to real NMH socket)
          → /tmp/claude-mcp-browser-bridge-{host-user}/{NMH-PID}.sock
            → Chrome NMH → Chrome Extension → chrome.debugger API
```

### Protocol

Native Messaging framing on the Unix socket:
- **4 bytes** — message length (uint32, little-endian)
- **N bytes** — JSON payload (UTF-8)

Example command:
```json
{"method":"execute_tool","params":{"client_id":"claude-code","tool":"tabs_context_mcp","args":{"createIfEmpty":true}}}
```

The bridge does raw byte forwarding — no parsing or modification of messages.

### Key discovery: USER env var

Claude Code's MCP server resolves the socket directory name from the `USER` environment variable. Docker does NOT set `$USER` automatically from the Dockerfile `USER` directive. Without `ENV USER=claude` in the Dockerfile, the MCP server looks in `/tmp/claude-mcp-browser-bridge-unknown/` and never finds the bridge socket.

### Key discovery: Unix sockets don't work through OrbStack volume mounts

Socket files appear in `ls` but `connect()` returns `ECONNREFUSED`. This is why the TCP bridge is necessary.

### Key discovery: chrome-native-host is NOT used by MCP

The file `~/.claude/chrome/chrome-native-host` is created by Claude Code for Chrome's `connectNative()` API. The MCP server does NOT spawn it — it directly watches the socket directory. The entrypoint still creates this file because Claude Code expects it to exist.

### Key discovery: socket permissions must be strict

The MCP server requires the socket directory to be `0700` and the socket file to be `0600`. Without this, the MCP server refuses to connect. socat supports `mode=0600` option for the socket; the directory is set via `mkdir -m 700`.

### Key discovery: socat needs setsid in Docker

When socat runs as a background process in `entrypoint.sh`, Ctrl+C in the container terminal sends SIGINT to the entire process group, killing socat. Using `setsid` moves socat to its own session, protecting it from terminal signals.

## File overview

| File | Location | Purpose |
|------|----------|---------|
| `bridge-host.js` | Host | TCP server, connects to real NMH socket |
| `entrypoint.sh` | Container | Credentials, chrome-native-host, socat bridge |
| `Dockerfile` | Build | node:20-slim + socat, Claude CLI, user matching host UID |
| `docker-compose.yml` | Build | Sets env vars |
| `Makefile` | Host | `up`, `shell` commands |

## Environment variables

| Variable | Where | Default | Purpose |
|----------|-------|---------|---------|
| `USER` | Dockerfile ENV | `claude` | Socket directory name |
| `BRIDGE_PORT` | docker-compose | `9229` | TCP port between bridges |
| `BRIDGE_TCP_HOST` | entrypoint.sh socat | `host.docker.internal` | Host address from container |
| `BRIDGE_USER` | bridge-host.js | `os.userInfo().username` | Host socket directory name |
| `BRIDGE_HOST` | bridge-host.js | `0.0.0.0` | TCP bind address |
| `CLAUDE_CREDENTIALS` | .env.local | — | OAuth JSON for ~/.claude/.credentials.json |

## Testing

To manually test the bridge from inside the container:

```bash
# 1. Verify socat is running and socket exists
ls /tmp/claude-mcp-browser-bridge-claude/

# 2. Test connectivity (requires bridge-host.js running on host)
node -e "
const net = require('net');
const fs = require('fs');
const dir = '/tmp/claude-mcp-browser-bridge-claude';
const sock = fs.readdirSync(dir)[0];
const s = net.createConnection(dir + '/' + sock, () => console.log('connected'));
s.on('error', e => console.log('error:', e.message));
s.on('close', () => console.log('closed'));
"

# 3. End-to-end test with NMH message (requires Chrome with extension)
node -e "
const net = require('net');
const fs = require('fs');
const dir = '/tmp/claude-mcp-browser-bridge-claude';
const sock = fs.readdirSync(dir)[0];
const s = net.createConnection(dir + '/' + sock, () => {
  const msg = Buffer.from(JSON.stringify({method:'execute_tool',params:{client_id:'test',tool:'tabs_context_mcp',args:{createIfEmpty:true}}}));
  const hdr = Buffer.alloc(4);
  hdr.writeUInt32LE(msg.length);
  s.write(Buffer.concat([hdr, msg]));
});
let buf = Buffer.alloc(0);
s.on('data', d => {
  buf = Buffer.concat([buf, d]);
  if (buf.length >= 4) {
    const len = buf.readUInt32LE(0);
    if (buf.length >= 4 + len) {
      console.log(JSON.parse(buf.slice(4, 4 + len).toString()));
      s.destroy();
    }
  }
});
s.on('error', e => console.log('error:', e.message));
"
```

## Debugging

- MCP logs: `~/.cache/claude-cli-nodejs/-home-claude/mcp-logs-claude-in-chrome/*.jsonl`
- bridge-host.js logs to stderr
- Use `strace -f -e trace=connect,openat -p <MCP_PID>` to trace MCP server (requires `SYS_PTRACE` capability in compose)

---
> Source: [vaclavpavek/claude-code-remote-chrome](https://github.com/vaclavpavek/claude-code-remote-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
