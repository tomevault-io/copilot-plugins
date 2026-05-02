## termhive

> A web-based management platform for coding CLI agents (Claude Code, Codex CLI, Gemini CLI). NOT a multi-agent coordination framework — this is a **human-driven dashboard** where the user manually manages multiple agent sessions organized by project teams.

# Termhive — Coding Agent Management Platform

## What This Is

A web-based management platform for coding CLI agents (Claude Code, Codex CLI, Gemini CLI). NOT a multi-agent coordination framework — this is a **human-driven dashboard** where the user manually manages multiple agent sessions organized by project teams.

Think of it as **tmux for coding agents** with a web UI, team organization, and shared content.

## Why It Exists

When running 3-7 coding agents simultaneously across different projects, the user currently:
- Has 7+ terminal windows open, can't find which is which
- Can't easily share context between agents (copy-paste between windows)
- Has no overview of what each agent is working on
- Can't manage agents from mobile/remote

## Core Architecture

```
┌─────────────────────────────────────────────┐
│              Web UI (React)                  │
│  ┌─────────┐ ┌─────────┐ ┌──────────────┐  │
│  │ Project  │ │ Agent   │ │   Shared     │  │
│  │ Sidebar  │ │Terminals│ │  Content     │  │
│  └─────────┘ └─────────┘ └──────────────┘  │
└──────────────────┬──────────────────────────┘
                   │ REST + WebSocket
┌──────────────────▼──────────────────────────┐
│           Express Server                     │
│  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │
│  │ PTY Mgr  │ │ Content  │ │  Project/   │ │
│  │(terminals)│ │  Store   │ │  Team Store │ │
│  └──────────┘ └──────────┘ └─────────────┘ │
└─────────────────────────────────────────────┘
```

## Tech Stack

- **Backend**: Node.js + Express + TypeScript
- **Frontend**: React + Vite + xterm.js (terminal rendering)
- **PTY**: node-pty (spawn real CLI processes)
- **Communication**: WebSocket (terminal I/O streaming) + REST (CRUD)
- **Storage**: JSON files on disk (no database needed)
- **Build**: tsup (backend) + vite (frontend)

## Data Model

```typescript
interface Project {
  id: string;
  name: string;           // e.g. "DexlessAI", "MedVault"
  description?: string;
  cwd: string;            // project root directory
  createdAt: string;
}

interface Agent {
  id: string;
  projectId: string;
  name: string;           // e.g. "Frontend", "Backend", "Alex"
  role?: string;          // optional label
  cli: 'claude' | 'codex' | 'gemini';
  cwd: string;            // working directory
  status: 'stopped' | 'running' | 'idle';
  pid?: number;
}

interface SharedContent {
  id: string;
  projectId: string;
  filename: string;       // e.g. "api-spec.md", "design-notes.md"
  content: string;
  createdBy: string;      // agent name or "user"
  updatedAt: string;
}
```

## Storage

All data stored in `~/.termhive/`:
```
~/.termhive/
├── config.json           # Global settings
├── projects/
│   ├── <project-id>/
│   │   ├── project.json  # Project metadata + agents list
│   │   └── content/      # Shared content files
│   │       ├── api-spec.md
│   │       └── design-notes.md
```

## Key Features (Priority Order)

### P0: Must Have
1. **Project Management** — CRUD projects with name, description, root directory
2. **Agent Terminals** — Spawn/stop/restart CLI agents (claude, codex, gemini) with real PTY
3. **Terminal Streaming** — xterm.js in browser, real-time I/O via WebSocket
4. **Agent Organization** — See all agents grouped by project, with status indicators
5. **Shared Content** — Per-project shared content store (create/read/update/delete markdown files)

### P1: Important
6. **Agent Input** — Type commands/prompts to any agent from the web UI
7. **Multi-terminal View** — Split view showing multiple agent terminals side by side
8. **Mobile Responsive** — Usable from phone (single-column layout)
9. **Auth** — Simple password protection (env var `AGENT_ORG_AUTH=user:pass`)

### P2: Nice to Have
10. **Agent Templates** — Save and reuse agent configurations
11. **Content Notifications** — Toast when shared content is updated
12. **Search** — Search across all shared content

## API Design

### REST Endpoints

```
# Projects
GET    /api/projects                    # List all projects
POST   /api/projects                    # Create project
PUT    /api/projects/:id                # Update project
DELETE /api/projects/:id                # Delete project

# Agents
GET    /api/projects/:id/agents         # List agents in project
POST   /api/projects/:id/agents         # Create agent
PUT    /api/projects/:id/agents/:aid    # Update agent
DELETE /api/projects/:id/agents/:aid    # Delete agent
POST   /api/projects/:id/agents/:aid/start   # Start agent (spawn PTY)
POST   /api/projects/:id/agents/:aid/stop    # Stop agent (kill PTY)
POST   /api/projects/:id/agents/:aid/restart # Restart agent

# Shared Content
GET    /api/projects/:id/content              # List shared content
GET    /api/projects/:id/content/:filename    # Read content
POST   /api/projects/:id/content              # Create content
PUT    /api/projects/:id/content/:filename    # Update content
DELETE /api/projects/:id/content/:filename    # Delete content
```

### WebSocket

```
ws://localhost:3200/ws

Client → Server:
  { type: "terminal:attach", agentId: "..." }     // Start receiving terminal output
  { type: "terminal:input", agentId: "...", data: "..." }  // Send input to terminal
  { type: "terminal:detach", agentId: "..." }      // Stop receiving
  { type: "terminal:resize", agentId: "...", cols: N, rows: N }

Server → Client:
  { type: "terminal:output", agentId: "...", data: "..." }  // Terminal output
  { type: "agent:status", agentId: "...", status: "..." }   // Status change
  { type: "content:updated", projectId: "...", filename: "..." }  // Content changed
```

## UI Layout

```
┌──────────────────────────────────────────────────────┐
│  Termhive                              [+ New Project]│
├────────────┬─────────────────────────────────────────┤
│            │                                         │
│  Projects  │   Agent Terminals (tabbed or split)     │
│            │  ┌─────────────────┬──────────────────┐ │
│  > DexlessAI│  │ Frontend (claude)│ Backend (codex) │ │
│    Frontend │  │ $ ...           │ $ ...            │ │
│    Backend  │  │                 │                  │ │
│    QA       │  │                 │                  │ │
│            │  └─────────────────┴──────────────────┘ │
│  > MedVault│                                         │
│    ...     │  ┌─────────────────────────────────────┐│
│            │  │ Shared Content          [+ New File] ││
│            │  │ api-spec.md | design.md | notes.md  ││
│            │  │ ┌─────────────────────────────────┐ ││
│            │  │ │ # API Spec                      │ ││
│            │  │ │ GET /todos → [...]               │ ││
│            │  │ └─────────────────────────────────┘ ││
│            │  └─────────────────────────────────────┘│
└────────────┴─────────────────────────────────────────┘
```

## What This Is NOT

- NOT a multi-agent coordination framework (no MCP, no contracts, no task lifecycle)
- NOT autonomous — the user manually tells each agent what to do
- NOT a harness — no system prompts, no role enforcement, no rate limiting
- Agents don't talk to each other — shared content is the only bridge, and the USER decides when to tell an agent to read/write it

## Reference

The UI/UX can reference vibehq-web (D:\agent-hub-cc\web\) for xterm.js terminal rendering patterns and WebSocket handling. The PTY management can reference vibehq's spawner (D:\agent-hub-cc\src\spawner\). But the architecture should be much simpler — no Hub, no MCP, no relay engine.

## Development

```bash
npm init -y
npm install express ws node-pty
npm install -D typescript tsup vite @types/express @types/ws react react-dom @xterm/xterm @xterm/addon-fit
```

Start with backend first (Express + PTY + WebSocket), then frontend (React + xterm.js).

---
> Source: [0x0funky/TermHive](https://github.com/0x0funky/TermHive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
