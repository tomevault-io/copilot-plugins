## hurlicane

> When asked to set up or install this project, run the following steps in order. Check each prerequisite and install if missing, then install npm dependencies.

# Claude Orchestrator

## Setup

When asked to set up or install this project, run the following steps in order. Check each prerequisite and install if missing, then install npm dependencies.

### 1. Check Node.js (requires ≥ 22)

```bash
node --version
```

If not installed or too old, install via [nvm](https://github.com/nvm-sh/nvm) or directly from nodejs.org:

```bash
# macOS with Homebrew
brew install node
```

### 2. Check tmux

```bash
which tmux || echo "MISSING: tmux not found"
```

Install if missing:

```bash
# macOS
brew install tmux
# Ubuntu/Debian
sudo apt install tmux
```

### 3. Check C++ build tools (required for node-pty)

```bash
# macOS
xcode-select -p || xcode-select --install
# Ubuntu/Debian
which make || sudo apt install build-essential python3
```

### 4. Check Claude Code CLI

```bash
which claude || echo "MISSING: claude not found"
```

Install if missing:

```bash
npm install -g @anthropic-ai/claude-code
```

### 5. Install npm dependencies

```bash
npm install
```

### 6. Verify setup

```bash
node --version   # should be ≥ 22
which tmux
which claude
ls node_modules | head -5
```

If everything looks good, start the server with:

```bash
npm run dev
```

Then open http://localhost:3000.

---

## Dev Commands

- `npm run dev` — start server (port 3000) + Vite dev server concurrently, both with hot reload
- `npm run build` — compile TypeScript and bundle client for production
- `npm run server:start` — run production build

## Architecture

- Server: Express + Socket.io on :3000; MCP SSE server on :3001
- Client: React 18 + Vite
- Database: SQLite via `node:sqlite` experimental (auto-created at `data/orchestrator.db`)
- Agents: spawned as `claude --print --output-format stream-json --verbose` subprocesses

## File Locking

A `PreToolUse` hook (`scripts/check-lock-hook.mjs`) blocks Edit/Write tool calls unless the agent holds a DB lock for that file. This applies to all Claude Code sessions running in this directory, including interactive ones.

To manually acquire a lock for direct edits while the orchestrator is running:

```bash
AGENT_ID="$ORCHESTRATOR_AGENT_ID"
FILE="/absolute/path/to/file"
LOCK_ID=$(node -e "const {randomUUID}=require('crypto');console.log(randomUUID())")
NOW=$(node -e "console.log(Date.now())")
EXPIRES=$(node -e "console.log(Date.now()+300000)")
sqlite3 data/orchestrator.db \
  "INSERT INTO file_locks (id,agent_id,file_path,reason,acquired_at,expires_at,released_at) VALUES ('$LOCK_ID','$AGENT_ID','$FILE','manual',$NOW,$EXPIRES,NULL);"

# ... make your edits ...

sqlite3 data/orchestrator.db \
  "UPDATE file_locks SET released_at=$(node -e 'console.log(Date.now())') WHERE id='$LOCK_ID';"
```

---
> Source: [lightsparkdev/hurlicane](https://github.com/lightsparkdev/hurlicane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
