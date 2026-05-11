## claude-code-image-sanitizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Image Sanitizer automatically resizes oversized images from screenshot tools (Puppeteer, Playwright, Chrome DevTools, etc.) to prevent the API error "image dimensions exceed max allowed size" which corrupts entire sessions.

**Available as:**
- **Plugin** (recommended) - Install via Claude Code marketplace
- **Standalone Hooks** - Install via `install.sh` script

**Two main features:**
1. **Image Resize Hooks** - Prevent future corruption by auto-resizing screenshots
2. **Session Repair Tool** - Fix already-corrupted sessions without losing work

## Installation & Testing

```bash
# Install dependencies (required: jq, imagemagick)
brew install jq imagemagick      # macOS
sudo apt install jq imagemagick  # Linux

# OPTION 1: Plugin Installation (Recommended)
# In Claude Code:
/plugin marketplace add MussaCharles/claude-code-image-sanitizer
/plugin install image-sanitizer@image-sanitizer-marketplace

# OPTION 2: Standalone Hooks Installation
./install.sh /path/to/project    # Per-project
./install.sh --global            # Global

# Uninstall standalone hooks
./uninstall.sh /path/to/project
./uninstall.sh --global

# Run all tests
./scripts/test.sh

# Run specific test suites
./scripts/test.sh resize      # Image resize tests only
./scripts/test.sh repair      # Session repair tests only
./scripts/test.sh install     # Installer tests only
./scripts/test.sh uninstall   # Uninstaller tests only
./scripts/test.sh plugin      # Plugin structure tests only

# Run tests with debug output
CLAUDE_IMAGE_DEBUG=1 ./scripts/test.sh

# Regenerate test fixtures
./scripts/test.sh --create-fixtures
```

## Architecture

### Standalone Hooks (`.claude/hooks/`)

**Three-hook system:**

1. **PreToolUse hook** (`resize-images.sh`): Intercepts screenshot tool calls, blocks fullPage screenshots that may exceed limits

2. **PostToolUse hook** (`post-screenshot-resize.sh`): The main fix - resizes captured images using ImageMagick if they exceed dimension/size limits

3. **Read hook** (`pre-read-resize.sh`): Resizes image files before they're read by the Read tool

**Hook pattern matching**: The shell scripts check for `screenshot|take_screenshot|puppeteer_screenshot` in the tool name. The broad matcher in settings.json triggers the hook, but the script filters to only process actual screenshot tools.

### Plugin (`plugin/`)

The plugin packages the same hooks for marketplace distribution:

```
plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── hooks/
│   └── hooks.json           # Hook definitions (uses ${CLAUDE_PLUGIN_ROOT})
├── scripts/
│   ├── resize-images.sh
│   ├── post-screenshot-resize.sh
│   ├── pre-read-resize.sh
│   └── repair-session.sh    # Session repair tool
├── commands/
│   ├── sanitizer-status.md  # /sanitizer-status slash command
│   └── sanitizer-repair.md  # /sanitizer-repair slash command
└── README.md
```

**Key difference**: Plugin hooks use `${CLAUDE_PLUGIN_ROOT}` instead of `${CLAUDE_PROJECT_DIR}` for paths.

### Documentation (`docs/`)

- `docs/HOOKS.md` - Detailed standalone hooks installation guide

### Session Repair Tool (`scripts/repair-session.sh`)

- Scans `~/.claude/projects/` for corrupted `.jsonl` files
- Detects corruption by finding lines >100KB (base64 images)
- Creates backups before modifying
- Replaces image data with placeholder text
- Removes `toolUseResult` to prevent UI crashes

## Configuration

Environment variables:
- `CLAUDE_IMAGE_MAX_DIMENSION` - Max pixels (default: 1200, safe for multi-image)
- `CLAUDE_IMAGE_MAX_SIZE_MB` - Max file size in MB (default: 4)
- `CLAUDE_IMAGE_QUALITY` - JPEG quality (default: 85)
- `CLAUDE_IMAGE_DEBUG` - Set to 1 for debug logging (logs to `/tmp/cc-image-sanitizer-debug.log`)
- `CLAUDE_FULLPAGE_ACTION` - "block" (default) or "warn" for fullPage screenshots

## Debug Logging

To see hook debug output:
```bash
export CLAUDE_IMAGE_DEBUG=1
claude

# View debug logs in real-time
tail -f /tmp/cc-image-sanitizer-debug.log
```

Logs are written to `/tmp/cc-image-sanitizer-debug.log` and prefixed with `[cc-image-sanitizer]`:
```
[2025-12-08 10:30:45] [cc-image-sanitizer] PostToolUse: mcp__playwright__browser_take_screenshot
[2025-12-08 10:30:45] [cc-image-sanitizer] Image: 1920x1080, 245678 bytes
[2025-12-08 10:30:45] [cc-image-sanitizer] Exceeds max dimension (1200)
[2025-12-08 10:30:46] [cc-image-sanitizer] Resized: 1920x1080 → 1200x675, 245678 → 98234 bytes
```

## Installer Behavior

The installer intelligently merges hooks:

| What | Behavior |
|------|----------|
| User's other hooks (Write, Bash, etc.) | **Preserved** |
| User's other settings (permissions, etc.) | **Preserved** |
| Image sanitizer hooks | **Replaced** with repo version |

**How it identifies our hooks:** By checking if `command` contains `resize-images.sh` or `post-screenshot-resize.sh`.

**Single source of truth:** Hook configuration is read from `.claude/settings.json` in this repo. To change the default matcher or timeout, edit that file and re-run the installer on target projects.

**Backup handling:** Creates backup before changes, auto-removes if no changes were made.

## Uninstaller Behavior

The uninstaller intelligently removes hooks:

| What | Behavior |
|------|----------|
| Image sanitizer hooks | **Removed** from settings.json |
| Hook script files | **Deleted** |
| User's other hooks | **Preserved** |
| User's other settings | **Preserved** |
| Empty hooks directory | **Removed** (if no other files) |

**Backup handling:** Creates backup before modifying settings.json.

## Key Constraints

- Anthropic limits: 8000x8000 max single image, **2000x2000 for multi-image (>20 images)**, 5MB max
- Default 1200px is safe for multi-image sessions
- ImageMagick handles both `magick` (v7+) and `convert` (v6) commands
- Hooks must remain bash-compatible for cross-platform support (macOS/Linux/WSL)

## Repair Tool Commands

```bash
./scripts/repair-session.sh                       # Interactive mode
./scripts/repair-session.sh --list                # List all sessions (sorted by time)
./scripts/repair-session.sh --find                # Find corrupted sessions
./scripts/repair-session.sh --fix <file>          # Fix specific session
./scripts/repair-session.sh --fix-all             # Fix all corrupted sessions
./scripts/repair-session.sh --delete-corrupted    # Delete all corrupted sessions
./scripts/repair-session.sh --list-backups        # List available backups
./scripts/repair-session.sh --restore <timestamp> # Restore a backup
./scripts/repair-session.sh --delete-backup <timestamp>  # Delete specific backup
./scripts/repair-session.sh --delete-all-backups  # Delete all backups
./scripts/repair-session.sh --help                # Show help
```

---
> Source: [MussaCharles/claude-code-image-sanitizer](https://github.com/MussaCharles/claude-code-image-sanitizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
