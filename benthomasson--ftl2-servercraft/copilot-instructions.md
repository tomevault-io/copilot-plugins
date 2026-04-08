## ftl2-servercraft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Game server lifecycle manager TUI built on FTL2. Manages Minecraft (NeoForge) and Terraria dedicated servers on Linode -- provisioning, monitoring, backup/restore, and automatic teardown of idle servers.

## Core Architecture

### Script Interface (`ftl2_servercraft/scripts/`)
- Each game type is a Python module implementing a standard interface: `provision`, `start_server`, `verify_server`, `restore_world`, `backup_world`, `get_player_count`, `save_and_stop`, `destroy`
- Scripts use FTL2's `automation()` context manager to run Ansible modules over SSH
- Game servers run in tmux sessions on the remote host; all monitoring and control happens through tmux commands
- `scripts/__init__.py` defines the interface contract and provides `get_script(name)` for dynamic loading

### TUI (`ftl2_servercraft/tui.py`)
- Textual app with a DataTable dashboard, activity log, and modal screens
- All long-running operations (provision, backup, stop, verify) run in background threads with their own asyncio event loops
- Shared `runtime_state` dict (written by threads, read by TUI timer) bridges thread state to the UI
- Provisioning logs are written to both the TUI and `logs/` directory

### Watchdog (`ftl2_servercraft/watchdog.py`)
- Polls player count at `poll_interval`, tracks empty time, auto-tears down after `grace_period`
- Handles consecutive connection failures (3+ = server down, trigger teardown)
- Always backs up the world before destroying the Linode

### Configuration (`ftl2_servercraft/config.py`)
- `ServerConfig` dataclass loaded from `servers.yml`
- `config_dir` points to a directory of game-specific config files (server.properties, run.sh, etc.)
- State files track provisioned Linode instances (IP, ID, label)
- `get_backups()` branches on `script` type for correct backup directory and file pattern

## Common Commands

```bash
# Run the TUI
ftl2-servercraft

# Check server status without TUI
ftl2-servercraft status

# Use a different config file
ftl2-servercraft --config my-servers.yml

# Install in development mode
uv pip install -e .
```

## Key Patterns

- **FTL2 automation context**: All remote operations happen inside `async with automation(...) as ftl:` blocks. The `ftl` object provides Ansible module access via attribute chains (e.g., `ftl.minecraft.shell(cmd=...)`, `ftl.terraria.dnf(name=...)`).
- **Host groups**: Terraria scripts use `ftl.terraria.*`, NeoForge scripts use `ftl.minecraft.*`. These correspond to Ansible inventory groups.
- **Secret bindings**: Environment variables (`LINODE_TOKEN`, `SLACK_TOKEN`) are mapped to specific module parameters via `secret_bindings` in the automation context.
- **Backup paths**: Minecraft backups go to `~/data/minecraft/backups/` as `.tar.gz`. Terraria backups go to `~/data/terraria/backups/` as plain `.wld` files.
- **Config files**: Game config files live in a separate repo/directory referenced by `config_dir` in servers.yml. They are copied to the remote server during provisioning.

## Adding a New Game Script

1. Create `ftl2_servercraft/scripts/<game>.py`
2. Implement all functions from the interface in `scripts/__init__.py`
3. Set `DESCRIPTION` string
4. Add a server entry in `servers.yml` with `script: <game>`
5. Update `get_backups()` in `config.py` if the backup directory/pattern differs

## Dependencies

- **ftl2** -- Automation framework (Ansible module execution over SSH)
- **textual** -- TUI framework
- **pyyaml** -- Config file parsing
- **linode_api4** -- Linode API client

## Related Repositories

- **ftl2** (`~/git/ftl2`): The underlying automation framework
- **server-configs** (external): Game-specific config files referenced by `config_dir`
- **ftl_builtin_modules** (`~/git/ftl_builtin_modules`): Ansible modules bundled by FTL2 for remote execution

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benthomasson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/benthomasson)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
