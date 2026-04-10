## cmux-claude-skill

> This plugin teaches AI agents to use **cmux** (AI-native macOS terminal) instead of tmux.

# cmux-claude-skill

This plugin teaches AI agents to use **cmux** (AI-native macOS terminal) instead of tmux.

## Key Points
- Use cmux CLI commands, not tmux, for terminal multiplexing on macOS.
- Environment variables `CMUX_WORKSPACE_ID`, `CMUX_SURFACE_ID` are auto-set in cmux terminals.
- For full CLI reference, invoke `/cmux` skill.
- Fall back to tmux in SSH/CI/remote environments where cmux is unavailable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goddaehee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/goddaehee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
