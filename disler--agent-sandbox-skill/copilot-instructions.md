## agent-sandbox-skill

> - Anytime the engineer starts a command with `\<prompt>`, look for the file and execute the command:

# Engineering Rules

## Executing Reusable Prompts

- Anytime the engineer starts a command with `\<prompt>`, look for the file and execute the command:
  - **Standard prompts**: `\<prompt>` maps to `.claude/commands/<prompt>.md`
  - **Nested prompts**: Use `:` as a path separator. Example: `\sandbox:host` maps to `.claude/commands/sandbox/host.md`
  - **Skill Prompts**: `\<prompt>` maps to `.claude/skills/<skill>/<prompt>.md`
  - **Agent sandbox prompts**: `\agent-sandboxes:<prompt>` maps to `.claude/skills/agent-sandboxes/prompts/<prompt>.md`
  - **Other Skills**: `\<skill>:<prompt>` maps to `.claude/skills/<skill>/<prompt>.md`
  - Properly fill in the variables in the command based on the `argument-hint` in the command file if it exists

---
> Source: [disler/agent-sandbox-skill](https://github.com/disler/agent-sandbox-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
