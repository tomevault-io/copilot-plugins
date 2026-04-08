## mcptools

> `mcptools` is a collection of tools exposed via the Model Context Protocol (MCP).

# Project Overview
`mcptools` is a collection of tools exposed via the Model Context Protocol (MCP).

## Current Tools
1.  **Trello Asset Downloader**: For downloading attachments from Trello cards.
2.  **Tmux Manager**: For managing tmux windows and panes from within an MCP environment.

# Tools

## Trello Asset Downloader
Located in `download_trello_asset/download_trello_asset.py`.
This tool allows downloading authenticated assets from Trello URLs.

### Dependencies
- Python 3
- `fastmcp`
- `python-dotenv`

### Configuration
The tool requires Trello API credentials. It supports loading them from environment variables or a `.env` file in the working directory (or installation directory).

Required variables:
- `TRELLO_API_KEY`
- `TRELLO_TOKEN`

### Running
To run the MCP server:
```bash
python3 download_trello_asset/download_trello_asset.py
```

## Tmux Manager
Located in `tmux_manager/tmux_manager.py`.
This tool allows listing windows and creating new windows in the current tmux session.

### Dependencies
- Python 3
- `fastmcp`
- `tmux` (must be installed and running)

### Functionality
- `tmux_list_windows`: Lists all windows in the current session.
- `tmux_new_window`: Opens a new window with a specified command.
- `tmux_capture_pane`: Captures text content from a pane.
- `tmux_split_window`: Splits a window into two panes.
- `tmux_select_window`: Switches to a specific window.
- `tmux_select_pane`: Focuses a specific pane.
- `tmux_kill_window`: Closes a window.
- `tmux_kill_pane`: Closes a pane.

**Session Management:**
- If the tool is running inside a tmux session, it operates on that session.
- If running outside tmux, it automatically creates and manages a dedicated session (`mcptools-session`), cleaning it up on exit if it created it.

# Installation
The project includes an `install.sh` script to install the tools locally.
```bash
./install.sh
```
This script will:
- Create a virtual environment.
- Install dependencies.
- Create a launcher script in `~/bin`.

# Development Conventions
- The project uses `fastmcp` to define tools.
- Tools are decorated with `@mcp.tool()`.
- Standard Python best practices and error handling are expected.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/satu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/satu)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
