## myscripts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of personal utility scripts for development workflows and system automation.

## Scripts

### Git AI Tools
- **git-ai**: Main entry point for AI-powered git operations. Currently supports `commit` subcommand.
- **git-ai-commit**: Generates commit messages using OpenAI API based on staged changes.
  - Requires `OPENAI_API_KEY` environment variable
  - Uses GPT-4o model with temperature 0.3
  - Prompts user for confirmation before committing
  - Prevents manual `-m/--message` flags to ensure AI generation

### Utilities
- **gyazo_local**: Screenshot capture tool that saves timestamped PNG files to ~/Downloads

## Development Notes

- All shell scripts use zsh (`#!/usr/bin/env zsh` or `#!/bin/bash`)
- Scripts are designed to be executable and placed in PATH
- Git AI tools have Japanese comments and error messages
- Error handling includes checking for required dependencies and environment variables

## Dependencies

- `curl` for API calls
- `jq` for JSON processing  
- `screencapture` (macOS) for screenshot functionality
- `OPENAI_API_KEY` environment variable for AI features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuuigara)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuuigara)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
