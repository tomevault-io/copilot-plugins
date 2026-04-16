## steroids-cli

> - Keep instructions modular and prefer progressive disclosure.

# Claude Code — project context
@./AGENTS.md

## Claude-specific notes
- Keep instructions modular and prefer progressive disclosure.
- If deeper scoped behavior is needed, use `.claude/rules/` files.
- Prefer single-responsibility functions. When touching a method that mixes multiple concerns, treat that as a prompt to split it along readability and responsibility boundaries before adding more logic.

## Debugging Protocol
- **Always check logs first** before using browser automation or code analysis. Look at server logs (`~/.steroids/logs/api.log`, `~/.steroids/logs/webui.log`), runner logs (`~/.steroids/runners/logs/`), and browser console errors before diving into source code.
- **Never manually trigger recovery.** Do not run `steroids monitor run`, `steroids runners wakeup`, reset tasks, or invoke any other recovery command to work around a bug. Fix the code so the system self-heals, then let the user restart and verify.
- **Monitor/recovery fixes require two test layers.** Any monitor, runner-health, recovery, or orphan-detection fix must add both targeted unit tests and a mock-style integration test that exercises the real DB shape and trigger path.

## How to Run Parallel Adversarial Reviews

When a design requires cross-provider review (see AGENTS.md for when), dispatch two reviewers **simultaneously** by sending a single message with two `Task` tool calls. Use whichever providers are available: Claude, Codex, Gemini, Mistral, or top reasoning models from Hugging Face.

**Call 1 — Claude reviewer:**
```
Task tool:
  subagent_type: superpowers:code-reviewer
  prompt: "Adversarial review of the following design. Look for technical debt,
           architectural regressions, type safety gaps, and logic holes. Be harsh.
           Do not suggest over-engineering. [paste full design doc content here]"
```

**Call 2 — alternate provider (pick whichever is available):**
```
Task tool:
  subagent_type: Bash
  prompt: "Run one of:
    timeout 1800 codex exec \"[adversarial review prompt]\"
    timeout 1800 gemini -p \"[adversarial review prompt]\"
    timeout 1800 mistral \"[adversarial review prompt]\""
```

Both calls go in a **single message** so they execute in parallel. When both return, assess each finding independently per AGENTS.md §"How to Conduct a Review" and append a Cross-Provider Review section to the design doc with adopt/defer/reject decisions for each finding.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UnlikeOtherAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
