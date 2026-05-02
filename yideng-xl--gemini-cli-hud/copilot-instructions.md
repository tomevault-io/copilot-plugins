## gemini-cli-hud

> This extension provides a real-time, bottom-sticky heads-up display (HUD) at the bottom of your terminal during Gemini CLI sessions.

# Gemini CLI HUD

This extension provides a real-time, bottom-sticky heads-up display (HUD) at the bottom of your terminal during Gemini CLI sessions.

## What It Shows

- **Active Model:** Current model name (e.g., `gemini-3-flash`)
- **Context Usage:** Progress bar with percentage and token counts (e.g., `Ctx: ████░░ 42% (420K/1.0M)`)
- **Tool Calls:** Claude-HUD style tool tracking (e.g., `✓ run_shell_command ×17 | ✓ replace ×1`)
- **GEMINI.md Count:** Number of loaded GEMINI.md files (project + global + extensions)
- **Extensions Count:** Number of installed Gemini CLI extensions
- **Active Skill:** Currently activated skill/extension
- **Session Timer:** Elapsed time since session start

## How It Works

The HUD uses DECSTBM (Set Top and Bottom Margins) to reserve the bottom rows of your terminal. Gemini CLI's content scrolls in the region above, while the HUD stays fixed at the bottom.

- **Hook** (`hook.js`): Invoked synchronously by Gemini CLI on each event. Renders HUD to `/dev/tty`.
- **Daemon** (`daemon.js`): Background process that maintains HUD state. Communicates with hook via Unix socket.

## Notes

- The HUD updates on each Gemini CLI event (SessionStart, AfterModel, AfterTool) — no background polling.
- If the terminal is resized, the HUD adjusts on the next event.
- Run `reset` after exiting Gemini CLI to clear the DECSTBM scroll region if needed.

---
> Source: [yideng-xl/gemini-cli-hud](https://github.com/yideng-xl/gemini-cli-hud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
