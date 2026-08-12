## voicesmith-mcp

> - **Never edit installed output files directly** (e.g., `~/.claude/CLAUDE.md`, `~/.cursor/rules/voicesmith.mdc`, `~/.codex/AGENTS.md`).

# VoiceSmith MCP — Project Rules

## Editing Rules
- **Never edit installed output files directly** (e.g., `~/.claude/CLAUDE.md`, `~/.cursor/rules/voicesmith.mdc`, `~/.codex/AGENTS.md`).
- Always edit the **template source** at `templates/voice-rules.md` instead.
- The install script (`install.sh`) renders templates with `{{MAIN_AGENT}}` substitution and injects them into user config files. The installed copies are derived artifacts — the template is the source of truth.
- After editing a template, update the local installed copy to match (or re-run the installer) so the current session reflects the changes.

---
> Source: [shshalom/voicesmith-mcp](https://github.com/shshalom/voicesmith-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
