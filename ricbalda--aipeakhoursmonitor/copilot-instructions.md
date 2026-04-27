## aipeakhoursmonitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal utility repository with two static HTML dashboards showing Claude AI peak usage hours (Mon-Fri 13:00-19:00 UTC), plus PowerShell scripts to switch between Anthropic and GLM/Z.AI proxy models for the Claude Code CLI.

No build system, no package manager, no tests, no git repo.

## Files

- **ClaudePeak1.html** — Full-featured dashboard: live status from `status.claude.com`, weekly timeline, browser notifications, dark/brand themes. Deployed to Cloudflare Pages (`claude-peak-time.pages.dev`).
- **ClaudePeak2.html** — Minimal card widget with promotional period tracking (Mar 13-28, 2026 "2x off-peak"). No external dependencies.
- **SetClaude.ps1** — Launches `claude` CLI with official Anthropic models (Haiku/Sonnet/Opus). Clears GLM env vars first.
- **SetGLM51.ps1** — Launches `claude` with GLM 5.1 proxy via environment variables (Z.AI endpoint).
- **SetGLM47.ps1** — Manages GLM proxy config in `~/.claude/settings.json` with `set`/`status`/`remove` commands. Creates timestamped backups before modifying.

## Key Details

- Both HTML files are fully self-contained (all CSS/JS inline). ClaudePeak1 loads Google Fonts and Cloudflare Analytics.
- Peak hours logic is duplicated in both HTML files — any change to the schedule (currently Mon-Fri 13:00-19:00 UTC) must be applied to both.
- PowerShell scripts are Windows-only and use Spanish comments.
- SetGLM scripts contain API tokens — treat as sensitive.
- No git history; changes are not tracked.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ricbalda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
