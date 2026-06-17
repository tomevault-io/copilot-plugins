## worktree-cleanup

> Automatically cleanup the worktree and return to the base branch. Keeps local and remote branches intact. Check after creating PR or pushing.


# Worktree Cleanup

Automatically cleans up the worktree after development is complete.

## Overview

This skill performs the following steps automatically:

1. Detects the current worktree directory and branch
2. Checks for uncommitted changes
3. Removes the worktree
4. Checks out the base branch (default: main)

**Important**: Local and remote branches are preserved (not deleted).

## Usage

### Basic Usage

```bash
# Run from within the worktree directory
bash scripts/worktree_cleanup.sh
```

### Options

```bash
# Specify a base branch
bash scripts/worktree_cleanup.sh --base develop

# Skip confirmation prompt
bash scripts/worktree_cleanup.sh --yes

# Ignore uncommitted changes (not recommended)
bash scripts/worktree_cleanup.sh --force
```

## Prerequisites

- Must be run from within a worktree directory
- All changes must be committed (or use the `--force` option)

## Error Handling

### Uncommitted Changes Detected

The script will stop and prompt you to commit. You can force execution with `--force`, but this is not recommended.

### Worktree Removal Failed

Manual removal instructions will be displayed.

### Running Outside a Worktree

A warning is displayed and the process is aborted.

## Details

See [REFERENCE.md](REFERENCE.md) for internal processing flow, troubleshooting, and best practices.

---
> Source: [gaebalai/worktree-cleanup](https://github.com/gaebalai/worktree-cleanup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
