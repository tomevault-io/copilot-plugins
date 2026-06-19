## js-knowledge-prism

> Pyramid-principle knowledge distillation — extract atoms, form groups, synthesize insights from journal notes.


# JS Knowledge Prism

A pyramid-principle-based knowledge distillation toolkit that transforms scattered journal notes into structured knowledge outputs.

## First Step: Detect Runtime Mode

Before performing any operation, detect whether this project is running as an **OpenClaw plugin** or in **standalone CLI mode**. The result determines configuration paths, command prefixes, and available features.

### Detection Steps

#### Step 0 — OS & Environment Variable Probe

First detect the current operating system to choose the correct shell commands, then check OpenClaw-related environment variables:

**OS Detection:**

| Check | Windows | macOS / Linux |
|-------|---------|---------------|
| OS identification | `echo %OS%` or `$env:OS` (PowerShell) | `uname -s` |
| Home directory | `%USERPROFILE%` | `$HOME` |
| Default OpenClaw state dir | `%USERPROFILE%\.openclaw\` | `~/.openclaw/` |
| Default config path | `%USERPROFILE%\.openclaw\openclaw.json` | `~/.openclaw/openclaw.json` |

**Environment Variable Check:**

```bash
# Windows (PowerShell)
Get-ChildItem Env: | Where-Object { $_.Name -match '^OPENCLAW_' }

# Windows (CMD / Git Bash)
set | grep -iE "^OPENCLAW_"

# macOS / Linux
env | grep -iE "^OPENCLAW_"
```

| Variable | Meaning if set |
|----------|---------------|
| `OPENCLAW_CONFIG_PATH` | Direct path to config file (e.g. `D:\.openclaw\openclaw.json`) — **highest priority**, use as-is |
| `OPENCLAW_STATE_DIR` | OpenClaw state directory (e.g. `D:\.openclaw`) — config file at `$OPENCLAW_STATE_DIR/openclaw.json` |
| `OPENCLAW_HOME` | Custom home directory (e.g. `D:\`) — state dir resolves to `$OPENCLAW_HOME/.openclaw/` |

**OpenClaw config file resolution order** (first match wins):

1. `OPENCLAW_CONFIG_PATH` is set → use that file directly
2. `OPENCLAW_STATE_DIR` is set → `$OPENCLAW_STATE_DIR/openclaw.json`
3. `OPENCLAW_HOME` is set → `$OPENCLAW_HOME/.openclaw/openclaw.json`
4. None set → default `~/.openclaw/openclaw.json` (Windows: `%USERPROFILE%\.openclaw\openclaw.json`)

Use the resolved config path in all subsequent steps.

#### Step 1 — OpenClaw Binary Detection

1. Check if `openclaw` command exists on PATH (Windows: `where openclaw`, macOS/Linux: `which openclaw`)
2. If exists, read the OpenClaw config file (path resolved by Step 0) and look for `js-knowledge-prism` in `plugins.entries` with `enabled: true`
3. Verify that `plugins.load.paths` contains a path pointing to this project's `openclaw-plugin/` directory

If **all three checks pass** → use **OpenClaw Plugin Mode**. Otherwise → use **Standalone CLI Mode**.

### Mode Comparison

| Aspect | OpenClaw Plugin Mode | Standalone CLI Mode |
|--------|---------------------|-------------------|
| Configuration | `~/.openclaw/openclaw.json` → `plugins.entries.js-knowledge-prism.config` | `.knowledgeprism.json` + `.env` |
| Command prefix | `openclaw prism <cmd>` | `npx js-knowledge-prism <cmd>` |
| AI tools | `knowledge_prism_*` (18 tools via OpenClaw Agent) | Not available (use CLI) |
| Cron auto-processing | `openclaw prism setup-cron` / `setup-output-cron` | Not available |
| Register / batch | `knowledge_prism_register` / `process_all` / `output_all` | Not available |
| Web UI | `http://<host>/plugins/js-knowledge/prism/` | Not available |
| Runtime data | `.openclaw/prism-processor/` (registry, inbox, batch) | None |
| Memory sync | Automatic to `work_dir/memory-export/` | Not available |

### OpenClaw Plugin Mode

When the plugin is deployed:

- **CLI**: always use `openclaw prism ...` instead of `npx js-knowledge-prism ...`
- **AI tools**: prefer `knowledge_prism_*` tools when invoked from an OpenClaw Agent session
- **Config**: modify `~/.openclaw/openclaw.json` for API endpoints, model, cron intervals, etc.; do NOT edit `.knowledgeprism.json` for plugin-managed settings
- **Cron**: manage via `openclaw prism setup-cron` / `openclaw prism setup-output-cron`
- **Registration**: use `openclaw prism register <dir>` to add knowledge bases for batch auto-processing
- **Output bindings**: use `knowledge_prism_bind_output` to configure automatic output generation
- **Runtime data**: check `.openclaw/prism-processor/registry.json` for registered bases and binding state

### Standalone CLI Mode

When running without OpenClaw:

- **CLI**: use `npx js-knowledge-prism <cmd>`
- **Config**: `.knowledgeprism.json` for processing params, `.env` for API credentials (see environment variable table in README)
- **No cron / register / batch** features — run `process` and `output` manually
- **No AI tools** — all interaction through CLI commands

---

## Deployment Probe

After detecting the runtime mode, run the following diagnostic steps to build a complete picture of the local deployment. Execute these in order; skip remaining steps if an earlier step indicates OpenClaw is unavailable.

> **Prerequisite**: Step 0 (OS & Environment Variable Probe) from the Detection Steps above must have already been executed. Use the detected OS to choose correct commands, and use the resolved config path from Step 0.

### Step 1 — OpenClaw Availability

- Windows: `where openclaw` / macOS & Linux: `which openclaw`
- If found: `openclaw --version` to confirm the installed version

### Step 2 — Plugin Load Status

Read the OpenClaw config file (path resolved by Step 0) and check:

- `plugins.load.paths` — does it include a path pointing to this project's `openclaw-plugin/` directory?
- `plugins.entries["js-knowledge-prism"].enabled` — is the plugin enabled?
- `plugins.entries["js-knowledge-prism"].config` — extract `baseDir`, `api.baseUrl`, `api.model` for a quick config snapshot

### Step 3 — Cron Job Status

```bash
openclaw cron list --json
```

Look for two jobs by name:

| Job Name | Purpose |
|----------|---------|
| `prism-auto-process` | Periodically runs journal → atoms → groups → synthesis pipeline |
| `prism-auto-output` | Periodically generates outputs from structure changes |

If either job is missing, the corresponding auto-processing is not configured. See the Runbook section for setup instructions.

### Step 4 — Knowledge Base Registration & Bindings

```bash
openclaw prism registered --status --json
```

Or use the `knowledge_prism_list_registered` AI tool. Check:

- Which bases are registered and enabled/disabled
- `lastProcessedAt` — when each base was last processed
- Pending journals count (from `--status`)

Then check output bindings:

```bash
# AI tool (preferred)
knowledge_prism_list_output_bindings

# Or read directly
# {workspace}/.openclaw/prism-processor/registry.json → bases[].outputBindings
```

### Step 5 — Runtime File Health Check

Inspect `{workspace}/.openclaw/prism-processor/`:

| File / Directory | Healthy State | Unhealthy Signal |
|-----------------|---------------|-----------------|
| `registry.json` | Exists | Missing → no bases registered |
| `output-inbox.jsonl` | Empty or absent | Lines present → unprocessed change signals (output cron may be stalled) |
| `output-batch-*.json` | Absent | Present → previous output run did not complete (crash recovery pending) |
| `output-archive/` | Contains completed batches | Not a concern; safe to clean up for disk space |

---

## Config Files Map

| File | Typical Path | Purpose | How to Modify |
|------|-------------|---------|--------------|
| `openclaw.json` | `~/.openclaw/openclaw.json` | Main config: API endpoint, model, cron intervals, plugin registration | Edit JSON directly |
| `.knowledgeprism.json` | `{baseDir}/` | Per-base config: name, api, process params, output bindings | Generated by `openclaw prism init`; can edit manually |
| `registry.json` | `{workspace}/.openclaw/prism-processor/` | Registration table + bindings + failed KL records | Modify only via tools/CLI; do NOT edit by hand |
| `output-inbox.jsonl` | `{workspace}/.openclaw/prism-processor/` | Change signal queue (written by process_all, consumed by output cron) | Auto-managed; safe to truncate (equivalent to "no pending changes") |
| `output-batch-*.json` | `{workspace}/.openclaw/prism-processor/` | In-progress output batch for crash recovery | Auto-managed; deleting abandons the batch |
| `output-archive/` | `{workspace}/.openclaw/prism-processor/output-archive/` | Completed batch archive | Read-only; safe to delete for disk space |

`{workspace}` resolves from `openclaw.json`: `agents.defaults.workspace` → `agents.list[0].workspace` → `process.cwd()`.

`{baseDir}` resolves from `openclaw.json`: `plugins.entries["js-knowledge-prism"].config.baseDir`.

---

## Action Priority

When performing an operation, always prefer the highest-priority method available:

> **OpenClaw AI Tool → OpenClaw CLI (`openclaw prism ...`) → Standalone CLI (`npx js-knowledge-prism ...`) / file edit**

| Scenario | Preferred | Fallback | Last Resort |
|----------|-----------|----------|-------------|
| Incremental processing | `knowledge_prism_process` | `openclaw prism process` | `npx js-knowledge-prism process` |
| View status | `knowledge_prism_status` | `openclaw prism status --json` | `npx js-knowledge-prism status` |
| Generate output | `knowledge_prism_output` | `openclaw prism output` | `npx js-knowledge-prism output` |
| Apply rewrite | `knowledge_prism_rewrite` | `openclaw prism rewrite --style <name>` | same as fallback |
| Batch-process all bases | `knowledge_prism_process_all` | no CLI equivalent | run `process` per base |
| Batch-output all bases | `knowledge_prism_output_all` | no CLI equivalent | run `output` per base |
| Register a base | `knowledge_prism_register` | `openclaw prism register <dir>` | N/A |
| Bind output | `knowledge_prism_bind_output` | edit `.knowledgeprism.json` `output.bindings` | N/A |
| Manage cron | `openclaw prism setup-cron` / `setup-output-cron` | `openclaw cron add/rm` manually | N/A |
| Change API / model | edit `~/.openclaw/openclaw.json` plugin config | edit `.knowledgeprism.json` + `.env` | N/A |
| View knowledge graph | `knowledge_prism_graph` | `openclaw prism graph` | open Web UI in browser |

---

## Runbook

### "Run output for me"

1. `knowledge_prism_list_output_bindings` — confirm bindings exist
2. Single base: `knowledge_prism_output --perspective <dir> --template <name>`
3. All bases: `knowledge_prism_output_all`

### "Cron doesn't seem to be running"

1. `openclaw cron list --json` — check if jobs exist
2. Missing → `openclaw prism setup-cron` and/or `openclaw prism setup-output-cron`
3. Present → `openclaw prism registered --status` — compare `lastProcessedAt` to expected interval to see if it's stalled

### "Switch model / API endpoint"

1. Edit `~/.openclaw/openclaw.json` → `plugins.entries["js-knowledge-prism"].config.api` (change `baseUrl`, `model`, or `apiKey`)
2. No restart needed — next cron trigger or manual run picks up the new config automatically
3. Standalone mode: edit `.knowledgeprism.json` or set `KNOWLEDGE_PRISM_API_*` environment variables

### "Add a new knowledge base"

1. `openclaw prism init <dir>` — scaffold the directory structure
2. `knowledge_prism_register` (or `openclaw prism register <dir>`) — add to auto-processing
3. Verify cron is configured; if not → `openclaw prism setup-cron`
4. Optional: `knowledge_prism_bind_output` — bind perspective + template for automatic output generation

### "Output is stuck / crash recovery"

1. Check `{workspace}/.openclaw/prism-processor/` for `output-batch-*.json`
2. Present → next `knowledge_prism_output_all` call resumes from the checkpoint automatically
3. To abandon the batch → delete the `output-batch-*.json` file; next run starts fresh from inbox
4. Inspect `registry.json` → `bases[].outputBindings[].failedKLs` to see which Key Lines are persistently failing

### "Clear backlog / reset state"

1. Truncate `output-inbox.jsonl` (equivalent to "no pending change signals")
2. Delete `output-batch-*.json` (abandons any in-progress batch)
3. `output-archive/` can be safely deleted (historical records only)

---

## What it does

Knowledge Prism processes journal notes through a three-layer pipeline:

1. **Atoms** — extract atomic knowledge units from daily journals
2. **Groups** — cluster related atoms into thematic groups
3. **Synthesis** — distill groups into top-level insights

Then uses these structured materials to generate perspective-driven outputs (articles, tutorials, etc.) via the SCQA + Key Line framework.

## Architecture

```
Journal Notes  →  Process Pipeline  →  Pyramid (atoms/groups/synthesis)
                                           ↓
                                    Perspectives (SCQA + Key Lines)
                                           ↓
                                    Outputs (articles, guides, etc.)
```

The OpenClaw plugin connects to an OpenAI-compatible LLM API to drive extraction and synthesis. All processing is incremental — only new journals are processed.

## Provided AI Tools

| Tool | Description |
|------|-------------|
| `knowledge_prism_process` | Run incremental pipeline (atoms → groups → synthesis) |
| `knowledge_prism_status` | Query knowledge base status and statistics |
| `knowledge_prism_graph` | Generate knowledge graph visualization HTML with statistics |
| `knowledge_prism_new_perspective` | Create a new perspective skeleton from template |
| `knowledge_prism_fill_perspective` | Generate SCQA or Key Line content for a perspective |
| `knowledge_prism_expand_kl` | Expand a Key Line into a full supporting argument document |
| `knowledge_prism_output` | Generate reader-facing output from a perspective |
| `knowledge_prism_rewrite` | Apply style rewrite to existing output files (single or batch) |
| `knowledge_prism_list_templates` | List available output templates |
| `knowledge_prism_list_rewrites` | List available rewrite definitions |
| `knowledge_prism_register` | Register a knowledge base for automatic cron processing |
| `knowledge_prism_unregister` | Remove a knowledge base from automatic processing |
| `knowledge_prism_list_registered` | List all registered knowledge bases with status |
| `knowledge_prism_process_all` | Batch process all registered bases; signals output inbox on changes |
| `knowledge_prism_bind_output` | Bind perspective+template for auto output generation (with `klStrategy` and optional `rewrites`) |
| `knowledge_prism_list_output_bindings` | List all output bindings and their status |
| `knowledge_prism_output_all` | Inbox/batch output with crash recovery and retry (structure refresh + change detection) |
| `knowledge_prism_discover_skills` | Query the extension skill registry |
| `knowledge_prism_install_skill` | Download and install an extension skill |

## CLI Commands

```
openclaw prism init <dir>              Initialize knowledge prism skeleton
openclaw prism process [--dry-run]     Run incremental processing
openclaw prism status [--json]         View processing status
openclaw prism graph [--base-dir]      Generate knowledge graph HTML
openclaw prism new-perspective <slug>  Create new perspective from template
openclaw prism output [--perspective]  Generate reader-facing output
openclaw prism rewrite --style <name>  Apply style rewrite to output files
openclaw prism register <dir>          Register knowledge base for auto-processing
openclaw prism unregister <dir>        Remove from auto-processing list
openclaw prism registered [--status]   List registered knowledge bases
openclaw prism setup-cron [--every N]  Configure cron job for automatic processing
openclaw prism setup-output-cron [--every N]  Configure cron job for auto output generation
openclaw prism sync [--force]          Sync knowledge base to memory system
```

## Web UI

The plugin registers HTTP routes on the OpenClaw gateway:

| Route | Description |
|-------|-------------|
| `/plugins/js-knowledge/prism/` | Hub page — lists all registered knowledge bases with graph status |
| `/plugins/js-knowledge/prism/api/bases.json` | JSON API — registry data with graph existence flags |
| `/plugins/js-knowledge/prism/graph/{index}` | Serves the generated `graph.html` for a registered base |

Access the hub at `http://<openclaw-host>/plugins/js-knowledge/prism/` after the plugin is loaded.

## Skill Bundle Structure

```
js-knowledge-prism/
├── SKILL.md                           ← Skill entry point (this file)
├── package.json                       ← Root package
├── LICENSE
├── openclaw-plugin/
│   ├── openclaw.plugin.json           ← Plugin manifest (config schema, UI hints)
│   ├── package.json                   ← ESM module descriptor
│   ├── index.mjs                      ← Plugin logic — 20 AI tools + CLI + HTTP routes
│   ├── output-inbox.jsonl             ← (runtime) change signals from process_all
│   ├── output-batch-*.json            ← (runtime) active batch for crash recovery
│   ├── output-archive/                ← (runtime) completed batch archive
│   └── skills/
│       ├── prism-processor/
│       │   └── SKILL.md               ← Cron auto-processing skill definition
│       ├── prism-template-author/
│       │   ├── SKILL.md               ← Output template creation guide (persona + style + type + prompt)
│       │   └── schema-reference.md    ← Complete frontmatter fields, variables, and section specs
│       └── prism-rewrite-author/
│           └── SKILL.md               ← Rewrite template creation guide
├── lib/
│   ├── config.mjs                     ← Configuration loading
│   ├── process.mjs                    ← Core pipeline (atoms → groups → synthesis)
│   ├── status.mjs                     ← Status collection
│   ├── graph.mjs                      ← Knowledge graph extraction and HTML generation
│   ├── init.mjs                       ← Project initialization
│   ├── new-perspective.mjs            ← Perspective creation
│   ├── fill-perspective.mjs           ← SCQA/Key Line generation
│   ├── expand-kl.mjs                  ← Key Line expansion
│   ├── output.mjs                     ← Reader-facing output generation
│   ├── rewrite.mjs                    ← Style rewrite engine (post-processing)
│   └── utils.mjs                      ← Shared utilities
└── templates/
    ├── graph.html                     ← 3D knowledge graph template
    ├── graph-hub.html                 ← Web UI hub page for multi-base graph overview
    └── outputs/
        ├── README.md                  ← Output architecture documentation
        ├── INDEX.md.tpl               ← Output index page template
        ├── components/
        │   ├── constraints.md         ← Global quality constraints (generic infrastructure)
        │   ├── review/base.md         ← Base review criteria (generic infrastructure)
        │   ├── persona/_scaffold.md   ← Persona component scaffold
        │   └── style/_scaffold.md     ← Style component scaffold
        ├── types/_scaffold.md         ← Type definition scaffold
        ├── prompts/_scaffold.md       ← Prompt template scaffold
        └── rewrites/_scaffold.md      ← Rewrite template scaffold
```

> `openclaw-plugin/index.mjs` imports from `../lib/` via relative paths, so the directory layout must be preserved.

## Prerequisites

- **Node.js** >= 18
- An **OpenAI-compatible API** endpoint (local or remote)

## Install

### Option A — One-command install (recommended)

**Linux / macOS:**

```bash
curl -fsSL https://raw.githubusercontent.com/user/js-knowledge-prism/main/install.sh | bash
```

**Windows (PowerShell):**

```powershell
irm https://raw.githubusercontent.com/user/js-knowledge-prism/main/install.ps1 | iex
```

By default, the skill is installed to `./skills/js-knowledge-prism`. To change:

```bash
curl -fsSL https://raw.githubusercontent.com/user/js-knowledge-prism/main/install.sh | JS_PRISM_DIR=~/.openclaw/skills bash
```

### Option B — Manual

1. Download the skill zip from GitHub Releases
2. Extract to your skills directory
3. Run `npm install` in the extracted directory
4. Register the plugin (see below)

### Register the plugin

Add to `~/.openclaw/openclaw.json`:

```json
{
  "plugins": {
    "load": {
      "paths": ["/path/to/js-knowledge-prism/openclaw-plugin"]
    },
    "entries": {
      "js-knowledge-prism": {
        "enabled": true,
        "config": {
          "baseDir": "/path/to/your-knowledge-base",
          "api": {
            "baseUrl": "http://localhost:8888/v1",
            "model": "your-model",
            "apiKey": "your-key"
          }
        }
      }
    }
  }
}
```

Restart OpenClaw to load the plugin.

## Plugin Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `baseDir` | string | cwd | Knowledge base root directory |
| `api.baseUrl` | string | `http://localhost:8888/v1` | OpenAI-compatible API endpoint |
| `api.model` | string | `default` | Model name |
| `api.apiKey` | string | `not-needed` | API key |
| `process.batchSize` | number | `5` | Journals per batch |
| `process.temperature` | number | `0.3` | LLM temperature |
| `process.maxTokens` | number | `8192` | Max tokens per request |
| `process.timeoutMs` | number | `1800000` | Request timeout (ms) |
| `cron.defaultInterval` | number | `60` | Auto-process interval (minutes) |
| `cron.outputInterval` | number | `120` | Auto-output interval (minutes) |
| `cron.timezone` | string | `Asia/Shanghai` | Cron timezone (IANA) |

## Verify

```bash
openclaw prism status
```

Expected output:

```
知识棱镜根目录: /path/to/base

  Journal: 12 篇 (5 个日期目录)
  Atoms:   24 个文件
  Groups:  6 个分组
  视角:    2 个
  待处理:  0 篇 journal
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `未找到 .knowledgeprism.json` | Not initialized | Run `openclaw prism init <dir>` |
| LLM timeout | API unreachable | Check `api.baseUrl` and network |
| Empty atoms | Journal has no extractable content | Ensure journals contain substantive notes |
| Tools not appearing | Plugin path wrong | Ensure path points to `openclaw-plugin/` subdirectory |

## Security

This skill only communicates with **user-configured** LLM API endpoints. It does not call any external APIs, collect telemetry, or transmit user data. All processing happens locally through the configured API.

## Extension Skills

Knowledge Prism includes bundled skills in `openclaw-plugin/skills/`:

| Skill | Description |
|-------|-------------|
| **prism-template-author** | Guide creation of output templates — persona, style, type, and prompt (four-layer scaffold + decision flow) |
| **prism-rewrite-author** | Guide creation of rewrite definitions (scaffold + style extraction from reference articles) |
| **prism-processor** | Cron auto-processing: batch pipeline + output generation with crash recovery |

All skills are bundled in `openclaw-plugin/skills/`.

> **Design principle**: The tool ships scaffolds and creation skills, not content-specific templates. All templates, types, components, and rewrite definitions are created by skills and stored in the knowledge base's `outputs/_templates/` directory.

## Links

- Source: https://github.com/user/js-knowledge-prism
- License: MIT

---
> Source: [imjszhang/js-knowledge-prism](https://github.com/imjszhang/js-knowledge-prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
