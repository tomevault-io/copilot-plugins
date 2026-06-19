## publisher

> Build, validate, and publish skills, plugins, agents, and apps to the NeboLoop marketplace. Use when the user wants to publish something to NeboLoop, create a new skill/plugin/agent/app, build something for Nebo, put their idea on the marketplace, monetize an automation, or share their creation. Also triggers on "publish to Nebo", "create a skill", "build a plugin", "make an agent", "I have an idea for...", "can I sell this on Nebo?".

# NeboLoop — From Idea to Marketplace

You are the user's publishing partner. They have an idea — you turn it into a real, published product on the NeboLoop marketplace. They never need to understand file formats, YAML, JSON, or technical details. You handle everything.

## Your Role

1. **Listen** — Understand what the user wants to create
2. **Decide** — Pick the right artifact type (skill, plugin, agent, or app)
3. **Build** — Generate all required files with correct structure
4. **Validate** — Check everything before publishing
5. **Publish** — Submit to NeboLoop marketplace automatically

The user says things like:
- "I want to build something that sends me a morning briefing"
- "Can I make a tool that connects to Stripe?"
- "I have an idea for a deal tracker"
- "Publish this to Nebo"

You respond by asking clarifying questions (if needed), then you build it and publish it. No manual steps. No config files. No tokens. No terminal commands for them to run.

## How Publishing Works (Behind the Scenes)

**Claude Desktop users:** You use the NeboLoop MCP tools directly. The user is already authenticated through their MCP connection. They don't need to do anything.

**Claude Code users:** You use the `neboai` CLI. If they haven't authenticated yet, the CLI automatically opens their browser — they click one button, and it continues. Zero friction.

**The user never needs to know which path you're using.** Just build it and publish it.

## Conversational Flow (Non-Technical Users)

When a user describes an idea without technical specifics, follow this flow:

**1. Understand the idea (1-2 questions max)**
- "What should it do?" (if unclear)
- "Who is this for?" (if it helps scope)
- Don't ask about file formats, languages, or architecture — decide those yourself

**2. Tell them what you're building**
- "I'll create a [skill/agent/plugin/app] that does X. Let me build that for you."
- Keep it one sentence. No technical details unless they ask.

**3. Build it silently**
- Generate all files in a working directory
- Follow all format rules in this skill
- Validate everything yourself

**4. Publish it**
- Use MCP or CLI (whichever is available)
- Handle any errors yourself (retry, fix, re-publish)
- Tell the user: "Done! Your [thing] is now on the NeboLoop marketplace."

**Never:**
- Ask which artifact type to use (you decide)
- Show them raw YAML or JSON (unless they ask)
- Ask them to run terminal commands
- Ask them to authenticate — it's automatic
- Explain the publishing process — just do it

---

## Artifact Hierarchy

```
APP  >  AGENT  >  SKILL
(UI)   (job)    (knowledge + actions)
```

Start with knowledge (Skill), compose into a job (Agent), add UI when chat isn't enough (App). Plugins are shared native binaries that skills depend on — they sit alongside the hierarchy.

## Language Preferences for Binaries

When generating code for plugins, sidecars, or any compiled binary:

| Priority | Language | Why |
|----------|----------|-----|
| 1 (preferred) | **Rust** | No runtime dependencies. Static binary. Does not trigger antivirus heuristics. Cannot be modified by the agent at runtime. Smallest attack surface. |
| 2 (acceptable) | Go | Static binary, fast compilation. Some AV false positives. Agent cannot modify at runtime. |
| 3 (avoid) | Python/Node | Interpreted — agent CAN modify scripts at runtime. Requires runtime installed. Larger attack surface. |

**Rust is strongly preferred** because:
- Single static binary per platform (no glibc, no runtime)
- AV-friendly: compiled Rust binaries don't trigger heuristic signatures
- Immutable: once compiled, the agent cannot alter the binary's behavior
- Cross-compilation is straightforward via `rustup target add`
- Small binary size with `--release` and `strip`

When scaffolding a new plugin or sidecar, use Rust `edition = "2024"` unless the user explicitly requests otherwise.

## What to Build — Decision Tree

| User Says | Build This |
|-----------|-----------|
| "teach the agent to..." / "when I say X, do Y" | **Skill** — markdown instructions |
| "connect to [service]" / "I need [API] access" | **Plugin** — native binary with tools + auth |
| "every morning..." / "monitor for..." / "when X happens, do Y" | **Agent** — workflows with triggers |
| "I need a dashboard" / "build me a [noun] tracker" | **App** — agent + frontend UI |
| "add a tool that..." / "give the agent the ability to..." | **Plugin** (if binary) or **Skill** (if instructions-only) |
| "connect an MCP server" / "wire in [an existing MCP]" / "publish this `mcpServers` block" | **Connector** — an installable MCP connection |
| "bundle these together" / "a starter pack of [skills/agents]" / "install all of these at once" | **Collection** — a bundle of existing artifacts |

---

## Building Skills

A skill is a folder with a `SKILL.md` that teaches the agent something. No manifest.json needed.

### Directory Structure

```
my-skill/
├── SKILL.md           # Required — YAML frontmatter + markdown body
├── scripts/           # Optional — executables the agent can run
├── references/        # Optional — detailed docs loaded on demand
├── assets/            # Optional — templates, images, fonts
├── examples/          # Optional — sample data
├── agents/            # Optional — sub-agent definitions
└── core/              # Optional — core logic files
```

These subdirectory names are conventions — the loader walks all files in the skill directory. Only hidden files, `SKILL.md`, `manifest.json`, and `signatures.json` are skipped during resource loading.

### SKILL.md Template

```yaml
---
name: my-skill-name
description: "[What it does] + [When to use it — include trigger phrases]"
version: "1.0.0"
triggers:
  - phrase one
  - phrase two
capabilities: []
plugins:
  - name: plugin-slug
    version: "*"
    optional: false
allowed-tools: Bash(some-binary *) Read Write
metadata:
  author: publisher-name
  secrets:
    - key: SERVICE_API_KEY
      label: "Service API Key"
      hint: "Get your key at https://service.example.com/keys"
      required: true
---
# Skill Title

[Imperative instructions. Step-by-step. Specific. Under 500 lines.]
```

### Frontmatter Fields

| Field | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `name` | yes | string | — | Lowercase letters, digits, and hyphens. 1-64 chars. Must not start/end with hyphen or contain `--`. |
| `description` | yes | string | — | What it does + when to use it. Max 1024 chars. Used by the LLM for routing decisions. |
| `version` | no | string | `"1.0.0"` | Semver version string. |
| `triggers` | no | string[] | `[]` | Case-insensitive substring phrases for programmatic activation. |
| `capabilities` | no | string[] | `[]` | Platform needs: `storage`, `network`, `vision`, `python`, `typescript`, `calendar`, `email`, `browser`. |
| `plugins` | no | object[] | `[]` | Plugin dependencies: `[{name, version, optional}]`. Version default: `"*"`. |
| `requires` | no | object[] | `[]` | Skill-to-skill dependencies: `[{name, version}]`. Version default: `"*"`. |
| `dependencies` | no | string[] | `[]` | Legacy skill dependencies (bare names). Use `requires` for new skills. |
| `allowed-tools` | no | string | `""` | Space-delimited pre-approved tool patterns (e.g., `Bash(my-bin *) Read Write`). |
| `priority` | no | integer | `0` | Higher = matched first when multiple skills match. |
| `max_turns` | no | integer | `0` | Max agent turns (0 = unlimited). |
| `platform` | no | string[] | `[]` | OS filter: `macos`, `linux`, `windows`. Empty = all platforms. |
| `license` | no | string | `""` | License identifier or file reference. |
| `compatibility` | no | string | `""` | Environment requirements description. Max 500 chars. |
| `author` | no | string | `""` | Publisher name. |
| `tags` | no | string[] | `[]` | Discovery tags. |
| `metadata` | no | object | `{}` | Arbitrary key-value pairs. Supports `secrets` array for secret declarations. |

**Important:** The `name` field does NOT need to match the directory name. The loader uses the YAML `name` as the key.

### Secret Declarations

Declare required secrets in `metadata.secrets`. Each entry has:

```yaml
metadata:
  secrets:
    - key: BRAVE_API_KEY        # Env var name, used in ${secret.BRAVE_API_KEY}
      label: "Brave Search Key" # UI label
      hint: "https://brave.com/search/api/"  # Help text or URL
      required: true
```

### Template Variables

| Variable | Resolves To |
|----------|-------------|
| `${NEBO_SKILL_DIR}` | Directory containing the SKILL.md |
| `${NEBO_DATA_DIR}` | Persistent data dir: `<data_dir>/appdata/skills/<name>/` (survives upgrades) |
| `${NEBO_USER_NAME}` | User's display name |
| `${NEBO_OS}` | `macos`, `linux`, or `windows` |
| `${NEBO_ARCH}` | `aarch64` or `x86_64` |
| `${plugin.SLUG_BIN}` | Resolved path to a plugin binary (slug uppercased, hyphens → underscores) |
| `${secret.KEY}` | Decrypted secret value from the `metadata.secrets` declaration |

Unrecognized `${...}` variables are preserved as-is.

### Writing Effective Skills

1. **Description is for the LLM.** The LLM reads the description to decide whether to activate the skill. Triggers are for programmatic substring matching. Both matter.
2. **Be imperative.** "Check the inbox" not "You should check the inbox."
3. **Be specific.** "7 days" not "recent". "$50,000" not "a significant amount."
4. **Decision rules over judgment.** "If >20%, highlight" not "highlight important changes."
5. **Show output format.** The agent needs to know what the result looks like.
6. **Error cases explicitly.** What to do when data is missing or the API fails.
7. **Under 500 lines.** Factor details into `references/` files.
8. **One skill, one job.** Don't combine unrelated responsibilities.

---

## Building Plugins

A plugin is a native binary that provides tools, hooks, commands, routes, providers, events, and config to the platform.

### Directory Structure

```
my-plugin/
├── PLUGIN.md          # Marketplace description
├── plugin.json        # Config: platforms, capabilities, auth, permissions
├── skills/            # Bundled skills that teach agents to use the tools
│   └── my-tool/
│       └── SKILL.md
└── dist/
    └── plugin/        # CLI reads binaries from dist/plugin/<platform>/
        ├── darwin-arm64/
        │   └── my-plugin
        └── linux-amd64/
            └── my-plugin
```

### Communication Protocol

Plugins communicate via **stdin/stdout**, not HTTP. The protocol depends on the capability type:

| Capability | Lifecycle | Input | Output |
|------------|-----------|-------|--------|
| **Tools** | One-shot subprocess | JSON on **stdin** | JSON on **stdout**, exit 0/1 |
| **Hooks** | One-shot subprocess | JSON payload on **stdin** | Modified JSON on **stdout** (filter hooks) or just exit (action hooks) |
| **Channel bridges** | Long-running process | NDJSON on **stdin** (outbound ops) | NDJSON on **stdout** (inbound messages). Must exit on stdin EOF. |
| **Event watches** | Long-running process | — | NDJSON on **stdout** (events). Multiplexed or single-event. |
| **Commands** | One-shot subprocess | Args from slash command | Text/JSON on **stdout** |
| **Routes** | One-shot subprocess | HTTP request proxied | HTTP response on **stdout** |

The binary receives CLI args from the `command` field in each capability definition (e.g., `"command": "gmail triage"`). For tools, the input JSON schema data arrives on **stdin**, not as CLI arguments.

### plugin.json

```json
{
  "id": "my-plugin",
  "slug": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "What this plugin provides",
  "author": "Publisher Name",
  "category": "connectors",
  "platforms": {
    "darwin-arm64": {
      "binaryName": "my-plugin",
      "sha256": "",
      "signature": "",
      "size": 0,
      "downloadUrl": ""
    }
  },
  "capabilities": {
    "tools": [],
    "hooks": [],
    "commands": [],
    "routes": [],
    "providers": [],
    "configSchema": []
  },
  "auth": {
    "type": "oauth_cli",
    "label": "Service Name",
    "description": "Connect your Service account",
    "commands": {
      "login": "auth login",
      "status": "auth status",
      "logout": "auth logout"
    },
    "env": {}
  },
  "permissions": {
    "envAllow": ["HOME", "PATH"],
    "network": true,
    "maxTimeoutSeconds": 300
  },
  "channel": null,
  "events": null,
  "dependencies": [],
  "triggers": [],
  "setup": null
}
```

**All plugin.json fields** (serde `camelCase`):

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | NeboAI artifact ID |
| `slug` | yes | URL-safe identifier |
| `name` | yes | Human-readable display name |
| `version` | yes | Semver version string |
| `description` | no | Brief description |
| `author` | no | Publisher name |
| `category` | no | Discovery routing category |
| `platforms` | yes | Map of platform key → `PlatformBinary` |
| `capabilities` | no | Structured capabilities (tools, hooks, commands, routes, providers, configSchema) |
| `auth` | no | Authentication configuration |
| `permissions` | no | Env access, network, timeout caps |
| `channel` | no | Channel bridge capability (for messaging integrations) |
| `events` | no | Event-producing watch capabilities |
| `dependencies` | no | Plugin-to-plugin dependencies: `[{name, version, optional}]` |
| `triggers` | no | Search matching keywords |
| `signingKeyId` | no | ED25519 signing key ID |
| `envVar` | no | Custom env var name override (default: `{SLUG}_BIN`) |
| `setup` | no | Multi-step setup wizard for guided configuration |

### Auth Fields

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | Auth type (e.g., `"oauth_cli"`, `"env"`) |
| `label` | no | Auth button label |
| `description` | no | User-facing description |
| `commands.login` | yes | Login CLI subcommand |
| `commands.status` | no | Status check CLI subcommand (returns JSON) |
| `commands.logout` | no | Logout CLI subcommand |
| `env` | no | Env vars to inject before auth commands |
| `help` | no | Help info: `{url, urlLabel, text}` |

### Tool Definition

```json
{
  "name": "service.action",
  "description": "What this does and WHEN to use it",
  "command": "service action",
  "inputSchema": {
    "type": "object",
    "properties": {
      "param": { "type": "string", "description": "What this is" }
    },
    "required": ["param"]
  },
  "approval": true,
  "timeoutSeconds": 120
}
```

- `approval` defaults to **`true`** — set `false` explicitly for read-only tools
- `timeoutSeconds` defaults to **120**
- Every property needs a `description`
- Use `enum` for fixed value sets
- Input JSON is sent to the binary on **stdin**, not as CLI args
- Always bundle skills that teach the agent *when* to use each tool

### Channel Plugins

For messaging integrations (Slack, Discord, Teams), use the `channel` field:

```json
{
  "channel": {
    "command": "bridge --listen",
    "name": "Slack",
    "description": "Connect to Slack workspaces",
    "restartDelaySecs": 5,
    "shared": false
  }
}
```

The bridge process is long-running. It receives outbound operations (reply, post, upload, dm) as NDJSON on **stdin** and emits inbound messages as NDJSON on **stdout**. The bridge MUST exit when stdin reaches EOF (orphan prevention). See [references/building-plugins.md](references/building-plugins.md) for the full NDJSON protocol.

### Rust Plugin Binary Pattern

```rust
use clap::{Parser, Subcommand};
use serde_json::json;
use std::io::{self, Read, IsTerminal};

#[derive(Parser)]
#[command(name = "my-plugin")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Auth {
        #[command(subcommand)]
        action: AuthAction,
    },
    /// Tool subcommand — reads input JSON from stdin
    Gmail {
        #[command(subcommand)]
        action: GmailAction,
    },
}

fn read_stdin_json() -> serde_json::Value {
    let mut buf = String::new();
    if !io::stdin().is_terminal() {
        io::stdin().read_to_string(&mut buf).ok();
    }
    if buf.is_empty() {
        json!({})
    } else {
        serde_json::from_str(&buf).unwrap_or(json!({}))
    }
}

fn main() {
    let cli = Cli::parse();
    match cli.command {
        Commands::Auth { action } => handle_auth(action),
        Commands::Gmail { action } => {
            let input = read_stdin_json();
            handle_gmail(action, &input);
        }
    }
}
```

**Output contract:** JSON object on stdout (success), JSON on stderr (error), exit code 0 (success) or 1 (error).

### Cross-Compilation (Rust)

```bash
# Cargo.toml: edition = "2024"
rustup target add aarch64-apple-darwin x86_64-apple-darwin
rustup target add aarch64-unknown-linux-musl x86_64-unknown-linux-musl

cargo build --release --target aarch64-apple-darwin
cargo build --release --target x86_64-unknown-linux-musl
strip target/aarch64-apple-darwin/release/my-plugin
```

Use `musl` for Linux — avoids glibc version issues.

### Valid Platforms

`darwin-arm64`, `darwin-amd64`, `linux-arm64`, `linux-amd64`, `windows-arm64`, `windows-amd64`

Minimum recommended: `darwin-arm64` + `linux-amd64` (a warning is logged if these are missing, but validation does not fail).

---

## Building Agents

An agent is a job description with workflows. Three files: `AGENT.md` (persona), `agent.json` (wiring), `manifest.json` (identity).

### Directory Structure

```
my-agent/
├── AGENT.md           # Persona — markdown, optional frontmatter
├── agent.json         # Workflows, triggers, tools, inputs, pricing
├── manifest.json      # Marketplace identity
└── skills/            # Optional — bundled skills
```

### AGENT.md

AGENT.md is the agent's persona — primarily prose. Frontmatter is optional and minimal:

```yaml
---
name: agent-name
description: One-line job description for marketplace discovery.
---
```

Only `name` and `description` are parsed from AGENT.md frontmatter. Everything else (triggers, workflows, version, pricing) belongs in `agent.json` or `manifest.json`.

```markdown
# Agent Name

[Who this agent is — personality, voice, approach]

## Communication Style
[How it talks: bullet points vs prose, formal vs casual, verbose vs terse]

## Capabilities
[What it can do — bulleted]

## Rules
[Hard constraints — never/always statements]

## Judgment
[Decision criteria for ambiguous situations]

## What You Don't Do
[Explicit boundaries]
```

### manifest.json

The JSON key for artifact type is `"type"`, not `"artifact_type"`:

```json
{
  "id": "my-agent",
  "name": "@org/agents/my-agent",
  "version": "1.0.0",
  "type": "agent",
  "description": "What this agent does",
  "author": "Publisher Name"
}
```

Additional manifest fields (all optional): `code`, `tags`, `runtime` (default `"local"`), `protocol` (default `"grpc"`), `signature`, `provides`, `permissions`, `implements`, `oauth`, `window`, `setup`.

### agent.json

```json
{
  "workflows": {
    "morning-briefing": {
      "trigger": { "type": "schedule", "cron": "0 7 * * 1-5" },
      "description": "Compile and deliver morning briefing",
      "activities": [
        {
          "id": "gather",
          "type": "research",
          "intent": "Collect overnight updates from all sources",
          "skills": ["@org/skills/news-reader"],
          "steps": ["Check RSS feeds", "Scan saved searches"],
          "token_budget": { "max": 4096 },
          "on_error": { "retry": 1, "fallback": "notify_owner" }
        }
      ],
      "budget": { "total_per_run": 6000 },
      "emit": "briefing.ready",
      "connections": []
    }
  },
  "requires": { "plugins": ["PLUG-XXXX-XXXX"] },
  "skills": ["@org/skills/name@^1.0.0"],
  "tools": [],
  "scopes": {},
  "inputs": [
    {
      "key": "timezone",
      "label": "Your Timezone",
      "type": "select",
      "required": false,
      "default": "US/Eastern",
      "options": [{ "value": "US/Eastern", "label": "Eastern" }]
    }
  ],
  "pricing": { "model": "monthly_fixed", "cost": 47.0 },
  "defaults": {
    "timezone": "user_local",
    "configurable": ["workflows.morning-briefing.trigger.cron"]
  },
  "memory": { "inherit_user": true, "context_isolated": false }
}
```

### Trigger Types

| Type | Use When | Fields |
|------|----------|--------|
| `schedule` | Fixed-time: "every weekday at 7am" | `cron` (5-field), `schedule` (optional human description) |
| `heartbeat` | Recurring interval: "every 30 min" | `interval` (e.g. `"30m"`), `window` (optional, e.g. `"08:00-18:00"`) |
| `event` | Reactive: "when X happens" | `sources` (pattern array, at least one required) |
| `watch` | Real-time plugin stream | `plugin`, `command` (supports `{{key}}`), `event` (optional), `restart_delay_secs` (default 5) |
| `folder` | Filesystem changes | `path` (supports `{{key}}`), `extensions` (e.g. `[".pdf", ".docx"]`), `recursive` (default true), `debounce_secs` (default 2) |
| `manual` | User-initiated only | — |

### Activity Fields

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `id` | yes | — | Unique within workflow |
| `intent` | yes | — | What the LLM should accomplish (imperative) |
| `type` | no | `""` | Activity type: `custom`, `research`, `email`, `notify`, `code`, `condition`, `loop`, `wait`, `agent`, `connector`, `http`, `transform` |
| `skills` | no | `[]` | Skill references to load for this activity |
| `mcps` | no | `[]` | MCP server references |
| `cmds` | no | `[]` | Workflow commands (e.g., `"emit"`) |
| `model` | no | `""` | Per-activity model override (e.g., `"sonnet"`, `"haiku"`, `"opus"`) |
| `steps` | no | `[]` | Step hints — the LLM has judgment |
| `params` | no | null | Arbitrary JSON parameters |
| `token_budget` | no | `{"max": 4096}` | Token budget for this activity |
| `on_error` | no | `{"retry": 1, "fallback": "notify_owner"}` | Error handling. Fallback: `notify_owner` (default), `skip`, `abort` |

Sum of `token_budget.max` values must ≤ `budget.total_per_run`.

### Input Fields

All inputs should have defaults (zero-config install). Template substitution: `{{key}}` in commands must match an input `key`.

| Type | Use For |
|------|---------|
| `text` | Short free-form (name, URL) |
| `textarea` | Long text (custom instructions) |
| `number` | Numeric (threshold, count) |
| `select` | Pick one from known set |
| `checkbox` | Boolean toggle |
| `radio` | Pick one with all visible |

Input options accept both object form `{"value": "x", "label": "X"}` and plain strings (auto-generates label from value).

---

## Building Apps

An app is an agent with a dedicated frontend UI. Use when chat output isn't enough.

### Directory Structure

```
my-app/
├── AGENT.md              # Persona
├── manifest.json         # Identity + type: "app" + permissions + window
├── agent.json            # Optional — workflows, skills, inputs, sidecar tools, scopes
├── ui/                   # Required — static frontend (served via neboapp:// protocol)
│   ├── index.html
│   ├── style.css
│   └── app.js
├── skills/               # Optional — teach agent how to use tools
│   └── workspace-mgmt/
│       └── SKILL.md
└── sidecar/              # Optional — Rust native backend (gRPC over Unix socket)
    ├── Cargo.toml
    └── src/main.rs
```

### manifest.json (App)

```json
{
  "id": "my-app",
  "name": "@org/agents/my-app",
  "version": "1.0.0",
  "type": "app",
  "description": "What this app does",
  "permissions": ["storage:readwrite", "subagent:invoke", "network:outbound"],
  "window": { "title": "My App", "width": 1024, "height": 768, "resizable": true }
}
```

Note: The JSON key is `"type"`, not `"artifact_type"`.

### Frontend SDK (`@neboai/app-sdk`)

```bash
pnpm add @neboai/app-sdk
```

```typescript
import { nebo } from '@neboai/app-sdk';

// Fetch — relative → sidecar, absolute → proxied through Nebo
const deals = await nebo.fetch('/deals').then(r => r.json());
const ext = await nebo.fetch('https://api.example.com/data').then(r => r.json());

// Storage — persistent async KV
await nebo.storage.setItem('key', value);
const val = await nebo.storage.getItem('key');
await nebo.storage.removeItem('key');

// Agents — invoke or stream
const { text, tools } = await nebo.agents.invoke('prompt');
for await (const chunk of nebo.agents.stream('prompt')) { ... }

// Janus — direct LLM (no persona)
const answer = await nebo.janus.complete({ messages: [...] });
for await (const text of nebo.janus.stream({ messages: [...] })) { ... }

// Chat — embedded UI panel
nebo.chat.mount(el, { placeholder: '...', theme: 'dark', contextId: id, scope: 'read' });
nebo.chat.send('message');
nebo.chat.onMessage((msg) => { ... });
nebo.chat.unmount();

// Surfaces — real-time agent→app events
nebo.surfaces.connect();
nebo.surfaces.on('state_snapshot', (e) => { appState = e.snapshot; render(); });
nebo.surfaces.on('state_delta', (e) => { render(); });
nebo.surfaces.on('run_started', (e) => showSpinner());
nebo.surfaces.on('run_finished', (e) => hideSpinner());
nebo.surfaces.send('action_name', { key: 'value' });

// WebSocket — auto-reconnecting
const ws = new nebo.WebSocket('/events');
ws.onmessage = (e) => console.log(e.data);

// Identity
const { id, name, model, skills } = await nebo.identity.get();
```

Full SDK reference: [references/building-apps.md](references/building-apps.md)

### Sidecar (Rust, gRPC over Unix Socket)

Sidecars communicate via **gRPC over a Unix socket**, not HTTP. The sidecar must implement the `UIService` gRPC service with a `HandleRequest` RPC.

**Environment variables provided to the sidecar:**

| Variable | Description |
|----------|-------------|
| `NEBO_APP_SOCK` | Unix socket path — bind gRPC server here |
| `NEBO_DATA_DIR` | Persistent storage directory |
| `NEBO_APP_TOKEN` | Per-launch auth token for API callbacks to Nebo |
| `NEBO_API_URL` | Nebo HTTP API base URL |
| `NEBO_APP_ID` | App/agent identifier |
| `NEBO_APP_NAME` | Human-readable app name |
| `NEBO_APP_VERSION` | App version string |
| `NEBO_APP_DIR` | App installation directory |

**Startup requirements:**
- Must start and bind the socket within 10 seconds (default; max 120s via `manifest.startup_timeout`)
- Handle SIGTERM gracefully (SIGKILL after 2s)

**Binary discovery order:** The runtime searches these locations: direct `binary` field → `app` binary → `tmp/` → `bin/` → `sidecar/target/release/`.

### Sidecar Tool Definitions

Tool definitions go in `agent.json`, NOT in a discovery endpoint. The system reads tool definitions from the `tools` array in `agent.json` at load time:

```json
{
  "tools": [
    { "name": "list_items", "description": "...", "method": "GET", "path": "/items" },
    { "name": "create_item", "description": "...", "method": "POST", "path": "/items",
      "input_schema": { "type": "object", "properties": { "name": { "type": "string" } } } }
  ]
}
```

### Tool Scopes

Restrict tools per embed context (works for agents and apps, not just apps):

```json
{
  "scopes": {
    "read": { "tools": ["list_items", "get_item"], "skills": [], "plugins": [] },
    "write": { "tools": ["list_items", "create_item", "delete_item"], "skills": [], "plugins": [] }
  }
}
```

SDK: `nebo.chat.mount(el, { scope: "read" });`

---

## Building Connectors

A connector is an installable **MCP connection** — its manifest *is* a standard MCP config block, the same `mcpServers` object Claude Desktop and VS Code use. Installing a connector wires that MCP server into the user's setup. Codes are `CONN-XXXX-XXXX`.

### Directory Structure

```
my-connector/
├── connector.json     # Required — the mcpServers block (+ optional metadata)
└── LISTING.md         # Optional — marketplace long description
```

### connector.json

The file must contain an `mcpServers` (or `servers`) object with at least one server; each server is either **stdio** (`command`) or **remote** (`url`). It may also carry `name`/`description`/`category`/`version`/`title` metadata alongside — the server validates only the `mcpServers` block and ignores the extra keys.

```json
{
  "name": "filesystem",
  "description": "Browse and edit local files from chat.",
  "category": "Build & connect",
  "version": "1.0.0",
  "mcpServers": {
    "fs": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"] }
  }
}
```

Remote example: `"acme": { "url": "https://mcp.acme.com/sse" }`.

**Publish:** `neboai publish ./my-connector` (auto-detected from `connector.json`). **MCP:** `connector(action: create, name: "...", manifestContent: "{...mcpServers...}")` then `connector(action: submit, id, version)`.

---

## Building Collections

A collection is a **bundle of existing artifacts** that installs as one unit — a "starter pack." Codes are `COLL-XXXX-XXXX`. A collection references other published artifacts by ID; it has no code of its own.

### collection.json

```json
{
  "name": "Sales Stack",
  "description": "Everything for outbound sales, installed in one paste.",
  "version": "1.0.0",
  "items": [
    { "targetId": "<skill-uuid>", "targetType": "skill" },
    { "targetId": "<agent-uuid>", "targetType": "agent" }
  ]
}
```

`items` is optional at create time (add them later). Each item needs a `targetId` (the artifact's UUID) and `targetType`. An optional `LISTING.md` sets the long description.

**Publish:** `neboai publish ./my-collection` creates the collection, adds each item, sets the listing, and submits when public. **MCP:** `collection(action: create, name)`, then `collection(action: add-item, id, targetId, targetType)`, then `collection(action: submit, id, version)`.

---

## Marketplace Listing

Every artifact carries **three text fields for three readers** — write them differently (full standard in `references/listing-quality.md`):

| Field | Read by | Source | Purpose |
|-------|---------|--------|---------|
| Manifest `description` (frontmatter) | the **LLM** | `SKILL.md`/`AGENT.md` frontmatter | trigger matching — keyword-rich is fine |
| Short `description` (≤500 chars) | a **human scanning a card** | frontmatter description (capped) | one benefit-led line |
| **`long_description`** | a **human on the detail page** | `LISTING.md` | the "What it does" sell |

**Display name vs id.** The marketplace **name** is human Title Case ("Nebo Design"); the **slug/id** stays lowercase-hyphen ("nebo-design"). The manifest frontmatter `name` must remain the lowercase id (it's the runtime id); set a clean display name with a frontmatter `title:` or let the tooling Title-Case the id.

**LISTING.md** is the publisher's marketing body — separate from the manifest the LLM reads. `neboai publish` reads it and sets the long description (and keeps it out of the runtime bundle). Via MCP, set it with `<type>(action: update, id, longDescription: "...")`. Write it benefits-first, in plain language, leading with the outcome — see `references/listing-quality.md`.

---

## Publishing — Implementation

Detect your environment and use the appropriate path. **Never ask the user to choose** — just detect and use.

### Detecting Your Path

Check your available tools:
- If any of these exist in your tool list → **use MCP path:**
  - `mcp__claude_ai_NeboLoop__skill`, `mcp__claude_ai_NeboLoop__agent`, `mcp__claude_ai_NeboLoop__plugin`, `mcp__claude_ai_NeboLoop__developer`
  - `mcp__levee__skill`, `mcp__levee__agent`, `mcp__levee__plugin`, `mcp__levee__developer`
- Otherwise → **use CLI path** (`neboai publish <directory>`)

Use `ToolSearch` to discover NeboLoop tools if unsure: search for "neboloop" or "levee".

### MCP Path (Claude Desktop / any MCP-connected environment)

The user is already authenticated. No auth step needed.

**Step 1: Select developer account (required for submission and binary uploads)**
```
developer(resource: account, action: select, id: "<developer-account-id>")
```
If no account exists, call `developer(resource: account, action: create, name: "Developer Name")` first.
Note: Creating private artifacts does NOT require a developer account — only submission to the marketplace does.

**Step 2: Create the artifact**

For skills:
```
skill(action: create, name: "my-skill-name", manifestContent: "<entire SKILL.md content>")
```

For agents:
```
agent(action: create, name: "my-agent-name", manifestContent: "<entire AGENT.md content>")
```

For plugins:
```
plugin(action: create, name: "my-plugin-name", category: "connectors")
```

**Step 3: Upload config and binaries (agents and plugins)**

Get an upload token:
```
skill(action: binary-token, id: "<ID>")
plugin(action: binary-token, id: "<ID>")
```

Then use the returned curl command to upload. The server reads only these multipart form fields:
- `file` — the binary (plugins and app sidecars; NOT `binary`)
- `config` — the config JSON (agent.json or plugin.json)
- `skills` — skills tarball (plugins, first platform upload only)
- `platform` — platform key (e.g., `darwin-arm64`); ignored for agents/apps

The manifest (SKILL.md / AGENT.md / PLUGIN.md) is **not** an upload field — it's set via `manifestContent` on create/update. Any `skill`/`manifest` form field is ignored by the server. For agents and apps the upload reads **only `config`** (no `platform`, no `file`).

Use `--http1.1` for large uploads to avoid HTTP/2 stream errors.

**Step 4: Submit for review**
```
skill(action: submit, id: "<ID>", version: "1.0.0")
agent(action: submit, id: "<ID>", version: "1.0.0")
plugin(action: submit, id: "<ID>", version: "1.0.0")
```

### CLI Path (Claude Code / Cursor / VS Code)

```bash
neboai publish <directory>
```

The CLI:
1. Detects artifact type from directory contents
2. Validates locally (structure, JSON, YAML, names, budgets)
3. Authenticates automatically (opens browser on first use)
4. Creates the artifact on NeboLoop
5. Gets upload token, uploads config + binaries per platform
6. Submits for review

**Override type:** `neboai publish ./dir --type agent`

**Visibility:** `neboai publish` defaults to `--visibility public` (submits for marketplace review). Use `--visibility private` (or `loop`) to publish unlisted without review — the CLI skips the submit step and tells the user it's unlisted.

**Listing:** the CLI also sets the marketplace listing automatically — a clean Title Case display name (from the frontmatter `title:`, else derived from the lowercase id) and, if a `LISTING.md` is present, the long "What it does" description. See the Marketplace Listing section.

**The user never runs auth commands.** Everything is automatic.

### What to Tell the User

After publishing succeeds:
- "Done! Your [skill/agent/plugin/app] has been submitted to the NeboLoop marketplace."
- "It'll be reviewed shortly. You can check its status anytime."
- Give them the artifact name and version

If publishing fails, diagnose and fix it yourself. Don't dump error messages on non-technical users.

---

## Validation

Always validate before publishing. Never skip this.

**If CLI is available:**
```bash
neboai validate <directory>
```

**If using MCP (no CLI):** Validate by checking these yourself:
- [ ] YAML frontmatter has `name` and `description`
- [ ] Name is lowercase + hyphens + digits only, 1-64 chars, no leading/trailing/consecutive hyphens
- [ ] JSON files parse cleanly (no trailing commas, no comments)
- [ ] No `{{template_vars}}` in plugin.json values — all must be literal
- [ ] Versions are valid semver (e.g., `"1.0.0"`)
- [ ] Budget math: sum of activity `token_budget.max` ≤ workflow `budget.total_per_run`
- [ ] Required files exist (SKILL.md for skills, AGENT.md + agent.json for agents)
- [ ] manifest.json uses `"type"` key, not `"artifact_type"`
- [ ] Sidecar tool definitions are in agent.json `tools` array

**If validation fails:** Fix it yourself. Don't ask the user to fix JSON or YAML — that's your job.

---

## Type Detection

| Present | Type |
|---------|------|
| `manifest.json` with `"type": "app"` | app |
| `plugin.json` | plugin |
| `connector.json` | connector |
| `collection.json` | collection |
| `agent.json` + `AGENT.md` | agent |
| `SKILL.md` only | skill |

Detection order (first match wins): app → plugin → connector → collection → agent → skill.

## Install Code Prefixes

Codes are Crockford Base32 (no I/L/O/U) in the format `PREFIX-XXXX-XXXX`.

| Prefix | Artifact |
|--------|----------|
| `SKIL-XXXX-XXXX` | Skill |
| `PLUG-XXXX-XXXX` | Plugin |
| `AGNT-XXXX-XXXX` | Agent |
| `APPS-XXXX-XXXX` | App |
| `CONN-XXXX-XXXX` | Connector |
| `COLL-XXXX-XXXX` | Collection |
| `WORK-XXXX-XXXX` | Workflow |
| `LOOP-XXXX-XXXX` | Loop |
| `NEBO-XXXX-XXXX` | Nebo |

App codes use the `APPS-` prefix end to end — the marketplace mints `APPS-` and the client install-code detector recognizes `APPS-`. They round-trip correctly.

## What Gets Uploaded

The manifest column is the content set via `manifestContent` on create/update — it is NOT an upload form field. Config and binary are the actual upload fields.

| Type | Manifest | Config | Binary | Skills Tarball |
|------|----------|--------|--------|----------------|
| Skill | SKILL.md | — | — | — |
| Plugin | PLUGIN.md | plugin.json | Per-platform binary | skills/ tarball |
| Agent | AGENT.md | agent.json (config only — no binary/platform) | — | — |
| App | AGENT.md | agent.json (config only) | Sidecar uploaded separately | — |
| Connector | connector.json (the `mcpServers` block) | — | — | — |
| Collection | — (created via `/collections`) | items added via `/collections/{id}/items` | — | — |

**Multi-file artifacts (bundle).** Any skill/agent/plugin can ship a directory — `references/`, `scripts/`, `assets/` alongside its manifest — as a single `.zip` to the bundle endpoint. `neboai publish` does this automatically; via MCP use `<type>(action: bundle-token, id)` to get a curl. The server extracts `SKILL.md`/`AGENT.md`/`PLUGIN.md` into the manifest and `agent.json`/`plugin.json` into config, and stores the rest as files that install with the artifact. (`LISTING.md` is the marketplace long description, not a runtime file — `neboai publish` reads it and sets it separately, and keeps it out of the bundle. See the Marketplace Listing section.)

## Managing Artifacts

When the user asks "what have I published?" or "check on my agent":

**MCP:** every type has its own tool with `list`/`get`/`create`/`update`/`delete`/`submit`:
```
skill(action: get, id: "<ID>")
plugin(action: get, id: "<ID>")
agent(action: get, id: "<ID>")
connector(action: get, id: "<ID>")     # MCP connections
collection(action: get, id: "<ID>")    # bundles of artifacts
marketplace(action: search, query: "search term")
```
Set the human listing on any of them: `<type>(action: update, id, name: "Title Case", longDescription: "...")`. Multi-file upload: `<type>(action: bundle-token, id)` returns a curl.

**CLI:**
```bash
neboai list                    # List published artifacts
neboai status <id>             # Check review status
neboai binaries list <id>      # List uploaded binaries
neboai binaries delete <id>    # Delete a binary (fix duplicates)
```

**Updating an artifact:** Use `action: update` with the same ID and new `manifestContent`. Then re-submit with bumped version.

---

## Critical Rules

1. **Config = agent.json.** NEVER upload manifest.json as the config field.
2. **Agent/app uploads read only `config`.** No `file`, no `platform` — the server's agent branch ignores both. (The CLI still sends `platform=linux-amd64`, but it's unused.)
3. **Plugin.json must be hardcoded.** No `{{template_vars}}` in plugin.json values.
4. **JSON must be valid.** No trailing commas. Validate: `python3 -c "import json; json.load(open('file.json'))"`
5. **Upload tokens expire in 5 minutes.** The CLI handles this automatically.
6. **HTTP/1.1 for uploads.** HTTP/2 causes stream errors on large files. CLI handles this.
7. **Skills tarball + config only on first platform upload** (plugins). Subsequent platforms are binary-only.
8. **Plugin binaries: recommend darwin-arm64 + linux-amd64.** Missing platforms log a warning.
9. **Duplicate version+platform = 500 error.** Delete existing binary first.
10. **Budget math must balance.** Sum of activity token_budget.max ≤ budget.total_per_run.
11. **manifest.json uses `"type"`, not `"artifact_type"`.** The serde rename makes the JSON key `"type"`.
12. **Sidecar = gRPC over Unix socket.** Not HTTP. Implement `UIService.HandleRequest`.
13. **Tools from agent.json, not discovery endpoint.** Sidecars do NOT serve `GET /_tools`.
14. **Plugin tools: input on stdin.** Not CLI arguments. Read JSON from stdin.
15. **on_error fallback default is `notify_owner`.** Not `skip` or `abort`.

---

## Error Recovery

Handle these yourself — never dump errors on the user.

| Error | Fix |
|-------|-----|
| Duplicate version+platform (500) | Delete existing binary, re-upload |
| Upload token expired | Get a fresh token and retry |
| Validation failed | Fix the issue in your generated files and retry |
| Auth failed (CLI) | The browser flow will retry automatically |
| Name already taken | Suggest a variant or ask the user for a new name |

**CLI tools:**
```bash
neboai binaries list <id>      # See what was uploaded
neboai binaries delete <id>    # Clean up partial uploads
```

**MCP:** Use `skill/agent/plugin(action: get, id: "<ID>")` to check state, then delete/recreate as needed.

---

## Building Reference Guides

For deep dives on each artifact type:

- [references/building-skills.md](references/building-skills.md) — Writing skills that trigger reliably and produce consistent results
- [references/building-plugins.md](references/building-plugins.md) — Binary architecture, tool design, events, auth, cross-platform builds
- [references/building-agents.md](references/building-agents.md) — Persona craft, workflow design, triggers, budgets, testing
- [references/building-apps.md](references/building-apps.md) — Frontend SDK, sidecar architecture, state management, tool discovery
- [references/listing-quality.md](references/listing-quality.md) — Writing a listing that gets installed: benefit-first description, plain-language inputs, name/category
- [references/review-rubric.md](references/review-rubric.md) — Reviewer gates (mechanical + human judgment) for approving submissions

## Format Quick Reference

- [references/skill-format.md](references/skill-format.md) — Complete SKILL.md field reference
- [references/plugin-format.md](references/plugin-format.md) — plugin.json schema and capabilities
- [references/agent-format.md](references/agent-format.md) — agent.json full field reference
- [references/app-format.md](references/app-format.md) — manifest.json, SDK, sidecar contract
- [references/connector-format.md](references/connector-format.md) — connector.json (mcpServers block) reference
- [references/collection-format.md](references/collection-format.md) — collection.json reference
- [references/common-mistakes.md](references/common-mistakes.md) — Every known gotcha with fixes

---
> Source: [NeboLoop/publisher](https://github.com/NeboLoop/publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
