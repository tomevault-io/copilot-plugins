## claudforge

> This document provides context and rules for AI agents (LLMs) contributing to the ClaudForge codebase.

# AI AGENT GUIDELINES — ClaudForge

This document provides context and rules for AI agents (LLMs) contributing to the ClaudForge codebase.

## 🏛 Core Architecture
- **Engine**: The core automation logic is codenamed **IRONCLAD**.
- **CLI Framework**: Built with `Typer` and `Rich`.
- **Automation**: Powered by `Playwright` (Chromium).
- **Persistence**: Global config is stored in `~/.claudforge/config.json`. Local project history is kept per-target.

## 🏷 Branding Rules (CRITICAL)
- **Engine Name**: Always use **IRONCLAD**. Do not substitute with "logic", "engine", "system", or any other codename.
- **Version Format**: Follow the pattern `vX.Y.Z IRONCLAD Engine` for all taglines and install blocks.
- **Identity**: IRONCLAD is a proper noun, not a descriptive term.

## 🛠 Engineering Principles
- **Cross-Platform**: Every feature must work on macOS, Windows, and Linux. avoid OS-specific libraries (e.g., AppleScript) in core modules.
- **Stealth**: Browser interactions must maintain "Human-Mimicry" (randomized delays, physical clicks).
- **Safety**: Always check for profile locks (`SingletonLock`) before launching a persistent context.
- **Metadata**: YAML parsing for `SKILL.md` must be robust (auto-quoting colons, etc.).

---
> Source: [VinuBalagopalAP/claudforge](https://github.com/VinuBalagopalAP/claudforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
