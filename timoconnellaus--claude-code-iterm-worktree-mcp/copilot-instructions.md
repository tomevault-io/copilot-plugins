## claude-code-iterm-worktree-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the MCP server directly (for testing)
python3 worktree_mcp_server.py

# Add to Claude Code MCP configuration
claude mcp add -s user worktree -- python3 path/to/worktree_mcp_server.py
```

## Architecture Overview

This is a Model Context Protocol (MCP) server that automates git worktree management with iTerm2 integration for Claude Code. The entire functionality is contained in a single Python file: `worktree_mcp_server.py`.

### Core Components

- **WorktreeMCPServer**: Main class that handles MCP protocol and worktree operations
- **iTerm2 Integration**: Uses the `iterm2` Python library for terminal automation
- **Git Worktree Management**: Creates, validates, and removes git worktrees
- **Metadata Tracking**: Maintains state files for managing active worktrees

### Key Features

1. **createWorktree**: Creates git worktree + branch, opens new iTerm tab, starts Claude session
2. **closeWorktree**: Validates cleanliness, removes worktree, closes tab, optionally deletes branch
3. **activeWorktrees**: Lists currently managed worktrees with tab IDs

### State Management

- `.worktree-manager-metadata.json`: Tracks active worktrees and their iTerm tab IDs (in parent directory)
- `.worktree-metadata.json`: Per-worktree metadata linking back to parent (in each worktree)
- Both files are automatically excluded from git via `.git/info/exclude`

### iTerm2 Automation Flow

When creating a worktree:
1. Creates new iTerm tab
2. Changes directory to worktree
3. Starts Claude with disallowed tools and the task description as argument
4. Switches back to original tab

### Validation Logic

- **Creation**: Checks git repo, branch existence, folder conflicts
- **Closure**: Verifies clean working tree, all commits pushed, or no commits on branch
- **Branch Deletion**: Automatically deletes branches with no commits beyond base branch

### Dependencies

- `iterm2`: For terminal automation (only works in iTerm)
- Standard library: `asyncio`, `json`, `subprocess`, etc.

### Important Notes

- Only provides tools when running in iTerm (detected via environment and API)
- Uses `--disallowedTools` to prevent worktree tool nesting in spawned Claude sessions
- Automatically handles git exclude setup for metadata files
- Parent directory structure: worktrees are created as siblings to the main repository

---
> Source: [timoconnellaus/claude-code-iterm-worktree-mcp](https://github.com/timoconnellaus/claude-code-iterm-worktree-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
