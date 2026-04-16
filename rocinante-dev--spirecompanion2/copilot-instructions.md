## spirecompanion2

> This project is an AI-powered companion application and mod for **Slay the Spire 2** (STS2).

# Spire Companion 2 — Gemini CLI Guide

This project is an AI-powered companion application and mod for **Slay the Spire 2** (STS2).

## Project Overview
- **Game Plugin (C#):** Extracts real-time game state to `state.json`.
- **Companion App (Python):** Serves a mobile-responsive dashboard and provides strategic advice.
- **Simulator (Python):** An immutable-state game engine for MCTS/RL training.

## Core Commands
Please refer to `CLAUDE.md` for the single source of truth regarding build, deploy, test, and run commands.

## Specialized Agent Skills
You have access to several project-specific skills in `.gemini/skills/`:
- `sts2-deployer`: Builds and syncs the mod to the local game folder for testing.
- `sts2-qa-sweeper`: Performs a pre-release audit to ensure no sensitive data is included.
- `sts2-releaser`: Finalizes packaging and uploads to GitHub.
- `sts2-simulator`: Guides development and testing of the game engine (see below).

## Development Conventions
- **Immutability:** The simulator engine strictly forbids in-place mutations. Use `replace()` for all state updates.
- **Thread Safety:** Godot/STS2 UI interactions in C# must use `CallDeferred()`.
- **Model Sync:** Always run `python tools/verify_models.py` after modifying the game state structure.

## Knowledge Base
- Game rules and data are located in `knowledge/master/`.
- Use the `sts2-librarian` MCP tools (`game_search`, `game_lookup_entity`) to query the raw C# source code for high-fidelity mechanical verification.

## Memories & Preferences
- Focus on "Visual & Card-Based" aesthetics for the companion UI.
- Prioritize "Approval-First" interactions for AI-driven advice.
- The simulator uses a "Blind Spot" protocol for auditing knowledge completeness.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rocinante-Dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
