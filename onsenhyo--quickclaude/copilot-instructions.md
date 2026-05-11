## quickclaude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

quickclaude is a CLI launcher for Claude Code. It scans `~/.claude/projects/` for project directories, presents an interactive selection menu, and spawns `claude` in the chosen directory.

## Commands

- **Run**: `node bin/quickclaude.js` or `npm start`
- **Install globally (for testing)**: `npm install -g .`
- **No build step or test suite** — this is a single-file Node.js CLI tool

## Architecture

The entire tool lives in `bin/quickclaude.js` (ESM, `#!/usr/bin/env node`):

- **`resolvePath(encoded)`** — Converts Claude's encoded project directory names (e.g., `-Users-dongwoo-projects-my-app`) back to real filesystem paths. Uses greedy longest-match against the filesystem, trying both `-` and space as segment joiners. Handles Windows drive letters (`C--Users-...`).
- **`getProjects()`** — Reads `~/.claude/projects/`, resolves each subdirectory name via `resolvePath`, filters out worktree dirs and non-existent paths.
- **`main()`** — Uses `@clack/prompts` for the interactive select UI, then spawns `claude` with `stdio: "inherit"` in the selected directory.

## Key Details

- Single dependency: `@clack/prompts` for terminal UI
- Published to npm as `quickclaude` (global install / npx)
- Requires Node.js 18+ and Claude Code installed

---
> Source: [onsenhyo/quickclaude](https://github.com/onsenhyo/quickclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
