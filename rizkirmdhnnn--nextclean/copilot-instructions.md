## nextclean

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nextclean is a single-file Go CLI tool that removes Next.js build artifacts (`.next`, `out`, `node_modules/.cache`) from a target directory, with optional recursive scanning.

## Build & Run

```bash
go build -o nextclean .
./nextclean <path>            # clean a single project
./nextclean -r <path>         # recursively find and clean all Next.js projects
./nextclean -r -cache <path>  # also remove node_modules/.cache
./nextclean -dry <path>       # preview what would be deleted
```

## Architecture

Single-file CLI (`main.go`) with no external dependencies. The flow is:
1. `parseFlags()` — CLI arg parsing via `flag` stdlib
2. `collectTargets()` — finds build artifact directories (non-recursive: checks known paths; recursive: uses `filepath.WalkDir`, skipping `node_modules` except for `.cache`)
3. Main loop — calculates sizes, deletes (or dry-runs), prints summary

---
> Source: [rizkirmdhnnn/nextclean](https://github.com/rizkirmdhnnn/nextclean) — distributed by [TomeVault](https://tomevault.io/claim/rizkirmdhnnn).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
