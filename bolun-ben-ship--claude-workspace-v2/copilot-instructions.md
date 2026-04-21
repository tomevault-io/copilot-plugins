## claude-workspace-v2

> The central workspace for RightClick:AI's SEO operations.

# RightClick:AI — Agency Workspace

## What This Is
The central workspace for RightClick:AI's SEO operations.
Contains the skill library, client registry, onboarding template, and agency-level outputs.

This is NOT a client workspace — open client subfolders in `clients/` separately as their own Claude Code project.

---

## Workspace Structure

```
RightClickAI-seo-workspace/
├── CLAUDE.md                          ← this file
├── clients.md                         ← client registry (all clients, platforms, status)
├── clients/                           ← one subfolder per client
│   └── {domain}/                      ← open THIS in Claude Code when working on a client
│       ├── CLAUDE.md                  ← client config (platform, handle, token env var)
│       ├── {client-slug}-*.json       ← Google service account key (per-client, gitignored)
│       ├── .claude/
│       │   └── commands/
│       │       ├── start-client.md    ← /start-client command (reference copy per client)
│       │       └── prime.md           ← /prime command (reference copy per client)
│       ├── context/
│       │   ├── client-info.md         ← brand, products, competitors, audience
│       │   ├── tone-guide.md          ← brand voice, writing rules (read by blog-write)
│       │   ├── current-data.md        ← live SEO coverage stats, analytics baseline
│       │   ├── strategy.md            ← content & growth strategy
│       │   └── personal-info.md       ← founder voice, values (optional)
│       ├── scripts/                   ← client automation scripts
│       │   └── mailchimp-sync.sh      ← Webflow + Calendly → Mailchimp lead sync (hourly)
│       ├── Content & SEO/
│       │   └── outputs/
│       │       └── {platform}-{handle}/
│       │           ├── audit/         ← AUDIT-YYYY-MM-DD.md
│       │           ├── implementation/← SEO-PLAN-YYYY-MM-DD.md, SNAPSHOT-YYYY-MM-DD.md
│       │           ├── research/      ← GSC-REPORT, GA4-REPORT, SOCIAL-TRENDS
│       │           ├── keywords/      ← KEYWORDS-YYYY-MM-DD.md
│       │           ├── blog-plans/    ← BLOG-PLAN-YYYY-MM-DD.md
│       │           └── blogs/         ← blog HTML posts
│       └── Design/
│           └── Carousel-YYYY-MM-DD/   ← /carousel output (HTML preview + PNG slides)
├── client-template/                   ← copy of this scaffolded by /onboard
│   ├── CLAUDE.md
│   ├── .claude/
│   │   └── commands/
│   │       ├── start-client.md        ← /start-client source (deployed globally by install.sh)
│   │       └── prime.md               ← /prime source (deployed globally by install.sh)
│   ├── context/
│   │   ├── client-info.md
│   │   ├── current-data.md
│   │   └── strategy.md
│   └── Content & SEO/outputs/
├── context/                           ← agency-level context
│   ├── agency-info.md                 ← what RightClick:AI does, skill stack
│   └── agency-strategy.md            ← agency goals and priorities
├── SKILLS-REFERENCE.md                ← master reference for all skills, agents, commands
├── outputs/                           ← agency-level implementation plans and reports
│   └── IMPLEMENTATION-PLAN-YYYY-MM-DD.md
├── seo-workflow/                      ← skill library (source of truth)
│   ├── install.sh                     ← deploys skills → ~/.claude/skills/, agents → ~/.claude/agents/, commands → ~/.claude/commands/
│   ├── agents/                        ← subagent configs
│   ├── carousel/SKILL.md             ← Instagram carousel generator
│   └── {skill-name}/SKILL.md         ← one folder per skill
└── .claude/
    └── commands/
        ├── start-agency.md            ← /start-agency command
        ├── onboard.md                 ← /onboard command
        └── initialise.md              ← /initialise command
```

---

## Skill Library — Source of Truth

`seo-workflow/` is the single source of truth for all skills:
- **Local use** → run `seo-workflow/install.sh` to deploy to `~/.claude/skills/`
- **GitHub** → commit and push `seo-workflow/` to version-control all skills
- **New machine** → pull repo → run `install.sh` → all skills ready

`install.sh` deploys three things:
1. **Skills** → `~/.claude/skills/` (24 skills)
2. **Agents** → `~/.claude/agents/` (6 subagents)
3. **Global commands** → `~/.claude/commands/` (`start-client.md`, `prime.md`)

Global commands are sourced from `client-template/.claude/commands/` and deployed to `~/.claude/commands/` so they appear in ALL Claude Code projects — including client subfolders that share this git root.

### Rules
- **Always edit skills in `seo-workflow/`** — never directly in `~/.claude/skills/`
- **Always edit commands in `client-template/.claude/commands/`** — never directly in `~/.claude/commands/`
- `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/` are deployment targets — overwritten on every `install.sh` run
- After any skill or command edit, run `bash seo-workflow/install.sh` to deploy

---

## Machine Setup — Global Permissions Required for Automation

The `/ai-seo-pipeline` skill creates Claude Code scheduled tasks that run unattended.
These tasks will pause and prompt for tool approval on every run unless the following
permissions are set globally in `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Bash", "Read", "Write", "Edit", "WebSearch", "WebFetch"]
  }
}
```

Add this once on any machine running scheduled SEO tasks. After adding, all pipeline
tasks (weekly blogs, monthly on-page, reports) will run fully autonomously.

If `/initialise` is run on a new machine, it should check for and set these permissions.

---

## MCP Server Configuration

**`settings.json` is NOT for MCP servers.** It handles env vars, permissions, and hooks only.
Putting `mcpServers` in `settings.json` will fail schema validation.

### Where MCP servers live

| Scope | Mac | Windows |
|---|---|---|
| **Global** (all projects) | `~/.mcp.json` | `%APPDATA%\Claude\mcp.json` |
| **Project-specific** | `.mcp.json` in the project folder | `.mcp.json` in the project folder |

### Rules
- **Global MCP servers** (e.g. PostHog, shared tools) → `~/.mcp.json` (Mac) or `%APPDATA%\Claude\mcp.json` (Windows)
- **Per-client MCP servers** (e.g. Webflow, Mailchimp for a specific client) → `.mcp.json` inside `clients/{domain}/`
- Auth tokens for MCP servers go in the `env` block of that same `.mcp.json`, referencing env vars
- The env var values themselves go in `~/.claude/settings.json` under `"env"` — that's the only MCP-related thing that goes in `settings.json`

### Example — global MCP (`~/.mcp.json` on Mac)
```json
{
  "mcpServers": {
    "posthog": {
      "command": "npx",
      "args": ["-y", "mcp-remote@latest", "https://mcp.posthog.com/sse", "--header", "Authorization:${POSTHOG_AUTH_HEADER}"],
      "env": { "POSTHOG_AUTH_HEADER": "Bearer phx_yourkey" }
    }
  }
}
```

### Example — project MCP (`.mcp.json` in client folder)
```json
{
  "mcpServers": {
    "webflow": {
      "command": "npx",
      "args": ["-y", "@webflow/mcp-server@latest"],
      "env": { "WEBFLOW_TOKEN": "${WEBFLOW_AEXPHL_TOKEN}" }
    }
  }
}
```

---

## Onboarding a New Client

Open this workspace in Claude Code and run:
```
/onboard domain.com
```
The command scrapes the site, auto-fills context files, scaffolds the client workspace,
updates `clients.md`, and tells you exactly what token to add to `~/.zshrc`.

---

## Commands

| Command | What it does |
|---|---|
| `/initialise` | New machine setup — deploys skills, checks all deps, env vars, credentials. Prints pass/fail report with fix steps |
| `/start-agency` | Loads agency context and produces an Agency Briefing — run at the start of every session |
| `/onboard domain.com` | Onboards a new client — scrapes site, scaffolds workspace, updates clients.md |

**Client workspace commands** (run from `clients/{domain}/` opened as a separate project):

| Command | What it does |
|---|---|
| `/prime` | Deep context loading — reads everything in full, produces a comprehensive Prime Brief for intensive work |
| `/start-client` | Loads all client context and produces a Client Briefing |
| `/3blog-seo-first-run` | Full run — audit → research → plan → write 3 blogs → approve → execute on-page changes → before/after report |
| `/ai-seo-pipeline` | Full long-term automation (3/6/12 months) — questionnaire → initial run → weekly blogs → monthly on-page → reports |
| `/seo-implementation-plan` | Global — builds a before/after implementation plan from any client workspace (plan only, no execution) |
| `/seo-final-report` | Global — produces a comprehensive end-of-engagement report comparing full history vs current state |
| `/carousel` | Instagram carousel generator — branded slides, HTML preview, export as 1080×1350px PNGs |

---

## Implementation Protocol (for changes to THIS workspace)

Before making any structural changes to the agency workspace itself (adding skills,
modifying the template, restructuring folders, updating install.sh):

1. Draft an implementation plan — what will change, why, what will NOT change
2. Save it to `outputs/IMPLEMENTATION-PLAN-YYYY-MM-DD.md`
3. Present a summary and wait for explicit approval
4. Execute after approval

Read-only tasks (reading files, checking structure, research) do not need a plan.

---

## Maintain This File

After ANY change to this workspace, check whether CLAUDE.md needs updating.
Update it immediately if any of the following have changed:

| Change | What to update |
|---|---|
| New `.claude/commands/` file added | Add it to the Commands table + SKILLS-REFERENCE.md |
| **Command logic changed** (any `.claude/commands/` file) | Update `SKILLS-REFERENCE.md` entry for that command + copy to ALL `clients/*/. claude/commands/` + run `install.sh` + confirm to user |
| New skill added to `seo-workflow/` | Add it to Workspace Structure diagram + SKILLS-REFERENCE.md + `seo-workflow/README.md`; run `install.sh` |
| Skill renamed or deleted | Update ALL of: SKILLS-REFERENCE.md, `seo-workflow/README.md`, `seo-workflow/install.sh` (SKILLS array + legacy list), all client CLAUDE.md files, client-template CLAUDE.md, client-template start-client.md, any orchestrator SKILL.md that references the old name in its Sub-skill Reference Paths section |
| Structural folder changes | Update the Workspace Structure diagram |
| New agency context files | Document them in `context/` section of the diagram |
| Client added/removed | Update `clients.md` (not this file — client data lives there) |
| Output filenames change in any skill | Update ALL of: `SKILLS-REFERENCE.md`, `seo-workflow/README.md`, and this file's Workspace Structure diagram |

**Rule: whenever a command is changed → update SKILLS-REFERENCE.md entry for that command, copy the updated file to ALL `clients/*/. claude/commands/` folders, run `bash seo-workflow/install.sh`, then CONFIRM to the user before closing the task.**

**Rule: whenever a skill is created, renamed, or significantly changed → update SKILLS-REFERENCE.md immediately, then run `bash seo-workflow/install.sh`, then CONFIRM to the user that SKILLS-REFERENCE.md has been updated before closing the task.**

**Rule: whenever output filenames or report structures change in a skill → grep `seo-workflow/` and the repo root for the old filenames before closing the task, fix every match, then commit all affected files together.**

**Rule: whenever a skill is renamed or deleted → grep the entire `seo-workflow/` directory for the old skill name and fix every match, including Sub-skill Reference Paths sections in orchestrator SKILL.md files. Also update `seo-workflow/README.md` Skill Index, Manual Install command, and Repo Structure diagram.**

This file is read by `/start-agency` to build the Agency Briefing.
Keeping it accurate means every new session starts with correct context.

---

## Credentials

Each client has their own isolated Google service account key:

- **File location:** `clients/{domain}/{client-slug}-*.json` (inside the client folder)
- **Env var:** `{CLIENT_SLUG_UPPER}_GOOGLE_KEY` → absolute path to the JSON file (set in `~/.zshrc`)
- **Naming example:** `AEXPHL_GOOGLE_KEY="$HOME/.../clients/aexphl/aexphl-seo-*.json"`
- **Never share keys across clients** — each client has their own service account
- JSON files are gitignored — never committed to the repo

On a new machine: copy each client's JSON key file to `clients/{domain}/` and add the env var to `~/.zshrc`.

### IMPORTANT: `~/.zshrc` alone is not enough

Claude Code's Bash tool does NOT source `~/.zshrc`. Env vars set only in `~/.zshrc` will show as `NOT SET` when Claude runs Bash commands.

**Every API key or token must be registered in TWO places:**

1. `~/.zshrc` — for your terminal/shell sessions
2. `~/.claude/settings.json` under the `env` block — for Claude Code

```json
{
  "env": {
    "MAILCHIMP_API_KEY": "your-key",
    "WEBFLOW_AEXPHL_TOKEN": "your-token"
  }
}
```

When onboarding a new client or adding any new API key, always update both files.

---

## API Key Security

**NEVER share API keys, tokens, or credentials in the chat.** If a key is accidentally shared:
1. Immediately go to the platform and revoke/regenerate it
2. Update `~/.zshrc` with the new key
3. Update `~/.claude/settings.json` env block with the new key
4. Restart Claude Code

This applies to all credentials: Mailchimp, Webflow, Google, Calendly, etc.

---

## Never
- Hardcode tokens or credentials in any output file or skill
- Share API keys or tokens in chat — rotate immediately if this happens
- Edit skills directly in `~/.claude/skills/` — always edit in `seo-workflow/` first
- Make structural changes without an implementation plan saved to `outputs/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bolun-ben-ship) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
