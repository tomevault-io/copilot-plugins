## sublime-sync

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
## Conversation Guidelines

- 常に日本語で会話する
## Project Purpose

This project aims to create a simple, frictionless backup and sync solution for Sublime Text 3, specifically designed for Windows environments where Sublime Text is preferred for its fast startup and simple behavior. The goal is to provide seamless backup and synchronization capabilities without requiring extensive technical setup, similar to how Tailscale provides effortless networking.

## Architecture Goals

- **Simplicity First**: Minimize technical preparation and configuration required by users
- **Windows-Optimized**: Designed primarily for Windows environments where Sublime Text 3 is commonly used
- **Frictionless Sync**: Automatic backup and synchronization between multiple machines
- **Minimal Setup**: Installation and configuration should be as simple as possible

## Development Context

This is a Python project managed within a UV workspace. The project is in early development stages, so architectural decisions should prioritize:

1. User experience over technical complexity
2. Reliability and data safety for user configurations
3. Cross-machine compatibility while maintaining Windows focus
4. Simple installation and onboarding process

## Common Commands

Since this project uses UV for dependency management within a workspace:

```bash
# Navigate to project directory
cd playground/sublime-sync

# Install dependencies (when they exist)
uv sync

# Run the application (when implemented)
uv run python -m sublime_sync
```

## Architecture Documentation

Detailed technical documentation is available in the `docs/` directory:

- `ARCHITECTURE.md` - Overall system architecture and data flow
- `docs/cloud-sync-design.md` - Cloud synchronization design and conflict resolution
- `docs/cloud-providers-comparison.md` - Cloud provider implementations and comparison
- `docs/data-flow-diagram.md` - Detailed data flow diagrams

## Implementation Notes

When implementing features, consider that the target users want minimal friction - they should be able to install and start syncing their Sublime Text configurations with as few steps as possible, similar to the Tailscale user experience they referenced as ideal.

### Cloud Sync Architecture

The system supports multiple cloud providers through an abstract provider interface:
- **Phase 1**: Dropbox (most mature API, excellent cross-platform support)
- **Phase 2**: Google Drive (high adoption, large free storage)  
- **Phase 3**: OneDrive (Windows integration)

Key design principles:
- **Conflict Resolution**: Smart merging with fallback to user choice
- **Data Safety**: Automatic backups before any destructive operations
- **Cross-Platform**: Consistent behavior across Windows, Mac, and Linux
- **Offline Resilience**: Graceful handling of network interruptions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tkys)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tkys)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
