## compass-skills

> This repository contains agent-agnostic `SKILL.md` skills. Use this file when your agent does not have native skills support, or when it needs explicit rules for discovering and applying the skills in `skills/`.

# COMPASS Agent Adapter

This repository contains agent-agnostic `SKILL.md` skills. Use this file when your agent does not have native skills support, or when it needs explicit rules for discovering and applying the skills in `skills/`.

## Discovery

- Available skills live under `skills/<skill-name>/SKILL.md`.
- Read each skill's YAML frontmatter `name` and `description` to decide whether it applies.
- Prefer explicit user invocation such as `$task-clarifier`, `$task-forest`, or `$user-profile-keeper`.
- If no skill clearly applies, continue normally; do not force a skill.

## Loading Protocol

When a skill applies:

1. Read the full `SKILL.md` before acting.
2. Resolve relative paths from the directory that contains that `SKILL.md`.
3. Read referenced files under `references/` only when the skill tells you to or when the current task needs them.
4. Run scripts from the skill's own `scripts/` directory instead of retyping large logic.
5. Keep generated runtime data out of the repository unless the user explicitly asks to commit it.

## Invocation Examples

```text
Use $task-clarifier to align this task before implementation.
Use $task-forest to update the task graph for this workspace.
Use $user-profile-keeper to initialize or update my local profile.
```

## Local Paths

- `task-forest` writes task data under the current workspace: `.agent-workbench/task-forest/`.
- `user-profile-keeper` writes profile data under the local user home: `.compass-skills/user-profiles/v1/`.
- Set `COMPASS_USER_PROFILE_HOME` to override the profile directory.
- Set `COMPASS_AGENT_NAME` to label task-forest changes by agent, for example `codex`, `claude-code`, `opencode`, or `openclaw`.

## Safety Rules

- Do not upload profile data, task data, credentials, cookies, browser sessions, or private local logs.
- Do not read password managers, keychains, SSH keys, browser cookies, API keys, tokens, or verification codes.
- Do not save secrets in user profiles or task graphs.
- Before destructive actions, publishing, remote writes, global configuration changes, credential use, or external side effects, ask for explicit user confirmation.
- Treat HTML exports as static offline views; they do not authorize modifying the task graph.

## Cross-Agent Notes

- Codex, Claude Code, OpenClaw, OpenCode, and similar local agents may expose different skill directories and invocation syntax.
- The portable contract is the file layout: `SKILL.md` plus optional `references/`, `scripts/`, `assets/`, and `agents/`.
- If your agent has no automatic skill router, use this `AGENTS.md` as the router policy.

---
> Source: [dongshuyan/compass-skills](https://github.com/dongshuyan/compass-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
