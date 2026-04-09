## quickgit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QuickGit (萌芽一键Git管理工具) is a Python CLI tool that simplifies Git operations through an interactive menu interface. It supports managing multiple Git repositories, handles both GitHub and Gitea remotes, and provides cross-platform compatibility (Windows, Linux, macOS).

## Running the Application

**Windows:**
```bash
run.bat              # Recommended - sets UTF-8 encoding
python quickgit.py   # Direct execution
```

**Linux/macOS:**
```bash
./run.sh             # Recommended (requires chmod +x run.sh first time)
python3 quickgit.py  # Direct execution
```

The tool is interactive - no command-line arguments needed. On startup, it prompts for a Git repository directory to manage.

## Architecture

The codebase follows a modular design with clear separation of concerns:

### Module Hierarchy

```
quickgit.py (entry point)
    └── ui.py (GitManagerUI)
        ├── git_operations.py (GitOperations)
        ├── remote_manager.py (RemoteManager)
        ├── utils.py (CommandExecutor, OutputFormatter, InputValidator, PlatformUtils)
        └── config.py (Config)
```

### Key Design Patterns

**1. Utility Classes Pattern** (`utils.py`)
- `CommandExecutor`: Centralized subprocess execution with UTF-8 handling
- `OutputFormatter`: Consistent colored console output (success/error/info/warning/step)
- `InputValidator`: User input validation and directory path handling
- `PlatformUtils`: Cross-platform operations (clear screen, path normalization, platform detection)
- `Colors`: ANSI color code definitions

**2. Operations Layer** (`git_operations.py`)
- `GitOperations` class manages all Git commands
- Maintains working directory state (`self.current_dir`)
- All Git operations use `CommandExecutor` for consistency
- Methods return `tuple[bool, str]` for success status and output

**3. Remote Management** (`remote_manager.py`)
- Handles GitHub and Gitea remote repository configuration
- GitHub format: `git@github.com:shumengya/{repo}.git`
- Gitea format: `ssh://git@git.shumengya.top:8022/{user}/{repo}.git`
- Manages multiple remotes per repository

**4. Configuration** (`config.py`)
- Centralized configuration in `Config` class
- Gitea server: `git.shumengya.top:8022`
- GitHub user: `shumengya`
- Default branch: `main`
- Includes comprehensive `.gitignore` template for Node.js, Go, Python projects

### Cross-Platform Considerations

The codebase handles platform differences through `PlatformUtils`:
- Path normalization with `os.path.expanduser()` and `os.path.normpath()`
- Platform detection: `is_windows()`, `is_linux()`, `is_mac()`
- Clear screen: `cls` (Windows) vs `clear` (Unix)
- Python command: `python` (Windows) vs `python3` (Unix)
- UTF-8 encoding explicitly set in subprocess calls

### State Management

- `GitOperations` maintains `self.current_dir` for the working directory
- Directory can be changed via `change_directory()` method
- All Git commands execute in the context of `self.current_dir`
- UI layer (`ui.py`) orchestrates state between operations and remote management

## Important Implementation Details

**Commit Message Format:**
- Default format: `update: YYYY-MM-DD HH:MM:SS` (see `git_operations.py:145`)
- Generated using `datetime.now().strftime('%Y-%m-%d %H:%M:%S')`

**Git Initialization Workflow:**
1. Run `git init`
2. Create `main` branch with `git checkout -b main`
3. Create `.gitignore` from template
4. Initial commit with message "first commit"

**Push Strategy:**
- First attempts direct push: `git push {remote} {branch}`
- If fails, retries with upstream: `git push -u {remote} {branch}`
- This handles first-time pushes to new remotes

**Subprocess Execution:**
- All commands use `shell=True` for compatibility
- UTF-8 encoding with `errors='ignore'` for robustness
- Captures both stdout and stderr combined
- Returns `(success: bool, output: str)` tuple

## Configuration Customization

To modify default settings, edit `quickgit/config.py`:
- `GITEA_HOST` and `GITEA_PORT`: Gitea server details
- `GITHUB_USER`: Default GitHub username
- `DEFAULT_BRANCH`: Default branch name (currently "main")
- `GITIGNORE_TEMPLATE`: Template for auto-generated .gitignore files

## Testing Considerations

When testing or modifying Git operations:
- Test with both existing and non-existing repositories
- Verify cross-platform path handling (Windows backslashes vs Unix forward slashes)
- Test with repositories that have/don't have remotes configured
- Verify UTF-8 handling for commit messages with non-ASCII characters
- Test the push retry logic (direct push → upstream push fallback)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shumengya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shumengya)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
