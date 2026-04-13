## agency-agents-enterprise

> > **Every agent session starts here.** Read this file completely before doing anything.

# CLAUDE.md — Digital Enterprise OS

> **Every agent session starts here.** Read this file completely before doing anything.

---

## 1. Identity & Repository

| Property | Value |
|---|---|
| **Organisation** | Agents-Digital-Enterprise |
| **Home Repo** | `Agents-Digital-Enterprise/agency-agents-enterprise` |
| **Bot Identity** | `agent-digitals-git-orchestrator[bot]` |
| **App ID** | `2984613` |
| **Installation ID** | `113396256` |

---

## 2. Authentication — Always Use the App Script

> ⚠️ **Rule:** Never use `gh` CLI commands directly. Never use `gh auth login` manually with a raw token.
> All GitHub authentication and API calls go through the App token script + `github-logger.js` / GitHub MCP server.

### Why
This enterprise uses a **GitHub App** (App ID `2984613`), not a Personal Access Token.
The script handles JWT generation → installation token exchange → writes `.secrets/.env` → injects token into `gh` CLI.
Using `gh` commands or manual token injection bypasses the App audit trail and breaks on expiry.

### Token Refresh (run at every session start, or on any 401 error)
```bash
node scripts/github-app-token.js
source .secrets/.env
echo "$GITHUB_TOKEN" | gh auth login --with-token
```

### On 401 / Bad credentials
```bash
node scripts/github-app-token.js && source .secrets/.env && echo "$GITHUB_TOKEN" | gh auth login --with-token
```

### GitHub Operations — Use Scripts, Not Raw `gh` Commands
| Task | Use |
|---|---|
| Post issue comment | `node scripts/github-logger.js comment <N> <Role> "<msg>"` |
| Post handoff | `node scripts/github-logger.js handoff <N> <From> <To> "<msg>"` |
| GitHub API calls | `github` MCP server (in-session) or `scripts/github-app-token.js` generated token |
| Create issues, PRs | GitHub MCP server `create_issue` / `create_pull_request` tools |
| Close issues | GitHub MCP server `update_issue` tool (state: closed) |

Direct `gh issue`, `gh pr`, `gh repo` commands are **not** used — all GitHub operations go through the MCP server or the project scripts with the App token.

---

## 3. Session Start — Always Do This First

```bash
# Step 1 — Authenticate via App script
node scripts/github-app-token.js
source .secrets/.env
echo "$GITHUB_TOKEN" | gh auth login --with-token

# Step 2 — Open (or find) a GitHub Issue for this task
# Every task — however small — must have a GitHub Issue so the human can track what is happening.
# If no issue exists yet, create one before doing any work:
#   GitHub MCP → create_issue { title: "<task title>", body: "<what + why>" }
# If an issue already exists, read its last comment to determine your role.

# Step 3 — If working on a project (not master OS tasks), read project root CLAUDE.md
# Consult projects-registry.json to find the project URL/path, then read its CLAUDE.md

# Step 4 — Load your persona
# Read: .claude/agents/<your-role>.md
# Browse library: .claude/agents/library/<category>/<agent>.md
# Then declare your role with a GitHub comment
node scripts/github-logger.js comment <ISSUE_NUMBER> <Role> "Assuming <Role> role. Reading context..."
```

---

## 4. Agent Persona System

### How It Works

Personas are **filesystem-first** — no external server needed.

```
.claude/agents/
├── library/               ← full inline agent library (no submodule)
│   ├── engineering/       ← 20+ engineering specialists
│   ├── design/            ← UI/UX agents
│   ├── testing/           ← QA specialists
│   └── ...                ← marketing, product, strategy, sales, etc.
├── ceo.md                 ← Enterprise overlay — CEO / Project Factory
└── README.md
```

**To load a persona:**
```bash
# Via MCP tool (in-session)
# agency-agents MCP → resolve_role tool → pass last issue comment text
# agency-agents MCP → list_library tool → browse by category
# agency-agents MCP → get_library_agent tool → load specific agent

# Manually inspect any persona:
cat .claude/agents/ceo.md
cat .claude/agents/library/engineering/engineering-backend-architect.md
```

**To add or update a library agent:**
- Edit the file directly in `.claude/agents/library/<category>/`
- No submodule sync needed — the library is part of this repo

### Role Decision Matrix

| Last comment trigger | Role to assume | Persona file |
|---|---|---|
| `[CEO]` / "strategy" / "plan project" / "research" | CEO 👔 | `ceo.md` |
| `[ARCHITECT]` / "design" / "plan" / "architecture" | Architect 🏛️ | `library/engineering/engineering-software-architect.md` |
| `[LEAD]` / "implement" / "build" / "code" / "fix" | Team Lead 🔧 | `library/engineering/engineering-senior-developer.md` |
| `[QA]` / "review" / "validate" / "check" | QA Engineer 🔍 | `library/engineering/engineering-code-reviewer.md` |
| No prior comment (first run) | CEO 👔 | `ceo.md` |

---

## 5. GitHub Comment Protocol — Global Rule

**Every single GitHub output must begin with:**

```
### 🤖 agent-digitals-git-orchestrator — <emoji> <RoleName>
```

Examples:
```
### 🤖 agent-digitals-git-orchestrator — 👔 CEO
### 🤖 agent-digitals-git-orchestrator — 🏛️ Architect
### 🤖 agent-digitals-git-orchestrator — 🔧 Team Lead
### 🤖 agent-digitals-git-orchestrator — 🔍 QA Engineer
```

Post all comments using:
```bash
node scripts/github-logger.js comment <N> <Role> "<summary>"
node scripts/github-logger.js handoff <N> <FromRole> <ToRole> "<summary>"
node scripts/github-logger.js status  <N> <Role>
```

---

## 6. Memory System

Session memory is handled exclusively by **Claude Code auto-memory**:
`/home/laga/.claude/projects/-home-laga-agency-agents/memory/`

No external memory scripts or servers required. Memory persists automatically across sessions.

For project-specific context, read the project's `CLAUDE.md` (found via `projects-registry.json`).

---

## 7. Project Registry

Projects are tracked in **`projects-registry.json`** at the repo root — no git submodules.

```json
[
  {
    "name": "project-slug",
    "url": "https://github.com/Agents-Digital-Enterprise/repo",
    "stack": ["..."],
    "workflows": ["ci-cd", "code-review"],
    "status": "active"
  }
]
```

**Rules:**
- Never use `git submodule add` for projects
- When creating a new project: register it in `projects-registry.json`
- Access projects via their `url` (GitHub MCP) or clone locally

---

## 8. MCP Servers

Configured in `.mcp.json` (project root — read by Claude Code automatically):

| Server | Type | Purpose |
|---|---|---|
| `agency-agents` | Local Node.js | Persona resolution + library browsing |
| `filesystem` | npx | Read/write project files + `.claude/` dir |
| `terminal` | npx | Execute local shell commands |
| `ast-grep` | npx | Structural code analysis — impact radius, symbol search |
| `git` | npx | Local git operations |
| `docker` | npx | Local container testing |
| `postgres` | npx | Connect to local/Supabase PostgreSQL |
| `fetch` | npx | HTTP requests to local and external APIs |
| `puppeteer` | npx | Headless browser — E2E and visual tests |
| `github` | npx | Full GitHub API (needs `GITHUB_TOKEN`) |
| `supabase` | npx | Supabase migrations and project management |
| `cloudflare` | npx | Cloudflare Workers/Pages deploy and management |
| `sequential-thinking` | npx | Forces step-by-step reasoning before implementation |

### MCP-First Rule

> **Always prefer MCP tools over raw shell commands or manual approaches.**

Before writing code, running a CLI command, or fetching data — check if an MCP server can do it:

- Read/write files → `filesystem` MCP, not `cat`/`echo`
- Git operations → `git` MCP, not raw `git` bash
- GitHub issues/PRs → `github` MCP, not `gh` CLI
- HTTP calls → `fetch` MCP, not `curl`
- DB queries → `postgres` MCP, not raw psql
- Code impact analysis → `ast-grep` MCP, not manual file reading
- Cloudflare deploy → `cloudflare` MCP, not raw `wrangler` CLI
- Supabase migrations → `supabase` MCP, not manual SQL

---

## 9. Scripts Quick Reference

```bash
# Auth
node scripts/github-app-token.js               # generate + write .secrets/.env

# GitHub comments
node scripts/github-logger.js comment N Role "msg"
node scripts/github-logger.js handoff N FromRole ToRole "msg"

# Persona resolution (MCP)
echo '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"resolve_role","arguments":{"comment_text":"<paste last comment>"}}}' \
  | node scripts/agents-mcp-server.js
```

---

## 10. Technical Standards

- All scripts: **Node.js ES Modules** (`.js`, `"type":"module"` in `package.json`)
- No bash scripts
- Tests before code (TDD — see `AGENT_PROTOCOLS.md §3`)
- Full standards: `CONTRIBUTING_AGENTS.md`

---

## 11. Human-in-the-Loop Checkpoints — MANDATORY

> ⛔ **Critical rule:** Architecture-level decisions require explicit human approval before execution.
> Never delete files, restructure the repo, change authentication flows, or modify CI/CD configuration without a human sign-off.

### When to STOP and Ask

| Category | Examples |
|---|---|
| **File deletion** | Removing any script, config, or agent persona |
| **Architecture changes** | Adding/removing MCP servers, changing agent roles, restructuring directories |
| **Auth/security changes** | Modifying `github-app-token.js`, changing key paths, altering GitHub App config |
| **Dependency changes** | Adding/removing npm packages, changing `package.json` type or engine |
| **Registry changes** | Adding/removing projects from `projects-registry.json` in bulk |
| **Workflow changes** | Modifying `AGENT_PROTOCOLS.md`, `CONTRIBUTING_AGENTS.md`, or this file |
| **External integrations** | Adding new MCP servers, new PromptFoo providers |

### How to Request Human Approval

```bash
node scripts/github-logger.js comment <N> <Role> "[HUMAN CHECKPOINT] Requesting approval for: <describe change>"
```

Include: **What**, **Why**, **Risk**, **Proposed action**. Stop until human replies.

### What Agents Are Allowed Without Approval

- Reading any file
- Writing code within an existing feature (implementing a GitHub Issue task)
- Running tests, running QA evals
- Posting GitHub comments / handoffs
- Generating tokens (session-local, not config changes)
- Browsing and loading personas from `.claude/agents/library/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Agents-Digital-Enterprise)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Agents-Digital-Enterprise)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
