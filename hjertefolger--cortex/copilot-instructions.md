## cortex

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Cortex is a Claude Code plugin that provides persistent local memory with cross-session recall. It uses vector embeddings and hybrid search to store and retrieve meaningful context from past sessions.

**Key Features:**
- Automated context management with configurable thresholds
- Smart compaction: save, clear, and restore continuity
- MCP server exposing memory tools to Claude
- Session analytics and insights
- Colored statusline with progress bar
- Granular memory saving with `cortex_remember`

## Build Commands

```bash
npm install            # Install dependencies
npm run build          # Build both index.js and mcp-server.js
npm run build:index    # Build main entry point only
npm run build:mcp      # Build MCP server only
npm run typecheck      # Type check without emitting

# Test with sample stdin data
echo '{"cwd":"/home/user/project","context_window":{"used_percentage":45}}' | node dist/index.js stats
```

## Architecture

### Data Flow

```
Claude Code → stdin JSON → parse → command router → handler → stdout
                                        ↓
                               SQLite + Embeddings
                                        ↓
                              ~/.cortex/memory.db

MCP Client → JSON-RPC → mcp-server.js → tools → database
```

### Core Components

| File | Purpose |
|------|---------|
| `src/index.ts` | Command router and handlers |
| `src/mcp-server.ts` | MCP server exposing tools |
| `src/stdin.ts` | Parse Claude Code's JSON input |
| `src/types.ts` | TypeScript interfaces |
| `src/database.ts` | SQLite schema, queries, FTS5 |
| `src/embeddings.ts` | BGE model loading, vector generation |
| `src/search.ts` | Hybrid search (vector + keyword + RRF) |
| `src/archive.ts` | Transcript parsing, content extraction, restoration context |
| `src/config.ts` | Configuration management |
| `src/analytics.ts` | Session tracking and insights |

### Database Schema

```sql
CREATE TABLE memories (
  id INTEGER PRIMARY KEY,
  content TEXT NOT NULL,
  content_hash TEXT UNIQUE,
  embedding BLOB NOT NULL,
  project_id TEXT,
  source_session TEXT,
  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE VIRTUAL TABLE memories_fts USING fts5(content);
```

### Search Algorithm

1. **Vector Search**: Query embedding vs stored embeddings (cosine similarity)
2. **Keyword Search**: FTS5 full-text search on content
3. **RRF Fusion**: Reciprocal Rank Fusion with k=60
4. **Recency Decay**: 7-day half-life for time weighting
5. **Result**: Top 5 sorted by combined score

### Stdin Format (Claude Code)

The plugin receives JSON via stdin from Claude Code:

```json
{
  "cwd": "/path/to/project",
  "transcript_path": "/path/to/session.jsonl",
  "model": {
    "id": "claude-opus-4-5-20251101",
    "display_name": "Opus"
  },
  "context_window": {
    "context_window_size": 200000,
    "used_percentage": 45
  }
}
```

## Plugin Structure

```
cortex/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata
├── .mcp.json            # MCP server configuration
├── skills/
│   ├── cortex-setup/SKILL.md     # Setup wizard (/cortex-setup)
│   ├── cortex-configure/SKILL.md # Configuration (/cortex-configure)
│   └── cortex-manage/SKILL.md    # Memory management (/cortex-manage)
├── hooks/
│   └── hooks.json       # SessionStart, PostToolUse, PreCompact
├── src/                 # TypeScript source
├── dist/
│   ├── index.js         # Compiled entry point
│   ├── mcp-server.js    # MCP server
│   └── sql-wasm.wasm    # SQLite WebAssembly
└── package.json
```

## MCP Tools

The MCP server exposes these tools:

| Tool | Purpose | Permission |
|------|---------|------------|
| `cortex_recall` | Search memory | Read-only |
| `cortex_remember` | Save specific insight/decision | Safe |
| `cortex_save` | Archive session (alias: cortex_archive) | Safe |
| `cortex_archive` | Archive session (canonical name) | Safe |
| `cortex_stats` | Get statistics | Read-only |
| `cortex_restore` | Get restoration context | Read-only |
| `cortex_update` | Update memory content or project | Safe |
| `cortex_rename_project` | Bulk move memories to new project | Safe |
| `cortex_delete` | Delete memory fragment | **Requires confirmation** |
| `cortex_forget_project` | Delete project memories | **Requires confirmation** |
| `cortex_analytics` | Get usage analytics | Read-only |

### Key Tool Distinction

- **`cortex_remember`**: Save a specific fact/decision during conversation. Does NOT require transcript path. Use for granular saving.
- **`cortex_save`/`cortex_archive`**: Archive entire session. Requires transcript path. Use for bulk session backup.

## Skills (User-Invocable Commands)

Skills are for multi-step workflows. Atomic operations (stats, recall, save) use MCP tools directly.

| Skill | Command | Purpose |
|-------|---------|---------|
| cortex-setup | `/cortex-setup` | First-time initialization |
| cortex-configure | `/cortex-configure` | Adjust settings |
| cortex-manage | `/cortex-manage` | Delete/manage memories |

## Hooks

| Hook | Trigger | Handler | Purpose |
|------|---------|---------|---------|
| `SessionStart` | New session | `session-start` | Show memory count, start analytics |
| `PostToolUse` | After any tool | `context-check` | Monitor context, auto-save/clear |
| `PreCompact` | Before compaction | `smart-compact` | Save + restoration context |

## Configuration

Config file: `~/.cortex/config.json`

```json
{
  "statusline": {
    "enabled": true,
    "showFragments": true,
    "showLastArchive": true,
    "showContext": true,
    "contextWarningThreshold": 70
  },
  "archive": {
    "autoOnCompact": true,
    "projectScope": true,
    "minContentLength": 50
  },
  "monitor": {
    "tokenThreshold": 70
  },
  "automation": {
    "autoSaveThreshold": 70,
    "autoClearThreshold": 80,
    "autoClearEnabled": false,
    "restorationTokenBudget": 1000,
    "restorationMessageCount": 5
  },
  "setup": {
    "completed": true,
    "completedAt": "2024-01-15T10:30:00Z"
  },
  "awareness": {
    "enabled": false,
    "userName": null,
    "timezone": null
  }
}
```

### Automation Settings

- `autoSaveThreshold`: Context % to trigger auto-save (default: 70)
- `autoClearThreshold`: Context % to trigger auto-clear (default: 80)
- `autoClearEnabled`: Enable automatic context clear (default: false)
- `restorationTokenBudget`: Max tokens for restoration context (default: 1000)
- `restorationMessageCount`: Messages to restore after clear (default: 5)

## Setup Flow

On first run:
1. Run `/cortex-setup` to initialize
2. Setup wizard creates `~/.cortex` directory
3. Initializes database and downloads embedding model
4. Configures statusline in `~/.claude/settings.json`
5. Marks setup as complete

After setup:
- Statusline appears in Claude Code (requires restart)
- Use `/cortex-configure` to adjust settings
- Memory tools are available via MCP

## Analytics

Analytics are stored at `~/.cortex/analytics.json` and track:
- Session metrics (peak context, save points, recalls)
- Usage patterns
- Recommendations for optimization

## Dependencies

- **sql.js**: SQLite via WebAssembly (bundled)
- **@xenova/transformers**: ONNX embeddings (external)
- **@anthropic-ai/sdk**: API types (external)

## Release Checklist

1. **Version bump** — update ALL of these files:
   - `package.json`
   - `.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json`
   - `src/mcp-server.ts` (`serverInfo.version` in `handleInitialize`)
2. **Verify no stale versions** — `grep -r "OLD_VERSION" package.json .claude-plugin/ src/`
3. **Update CHANGELOG.md** — add entry at top
4. **Build** — `npm run build` (rebuilds dist/ with new version)
5. **Typecheck** — `npx tsc --noEmit`
6. **Commit** — `chore: release vX.Y.Z`
7. **Tag** — `git tag vX.Y.Z`
8. **Push** — `git push && git push --tags`
9. **GitHub release** — `gh release create vX.Y.Z`
   - **Check previous release first**: `gh release view <prev-tag>` and match its format
   - Must include the "Updating from v2.x" section with plugin update commands, setup instructions, restart note, and the marketplace-first warning link
   - Include "Full Changelog" compare link

## Development Notes

- Uses esbuild for bundling with external dependencies
- sql-wasm.wasm must be copied to dist/ during build
- Embedding model is downloaded on first use (~33MB)
- Database is persisted at ~/.cortex/memory.db
- Analytics are stored at ~/.cortex/analytics.json
- Tool Search is handled automatically by Claude Code (activates at 10% context usage)

---
> Source: [hjertefolger/cortex](https://github.com/hjertefolger/cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
