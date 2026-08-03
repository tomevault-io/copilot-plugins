## self-improving-claude

> [Describe what this project is and what Claude's role is. Be specific: what decisions should Claude make independently vs. escalate? What does "good work" look like here?]

# [Your Project Name] — Context for Claude

## Purpose
[Describe what this project is and what Claude's role is. Be specific: what decisions should Claude make independently vs. escalate? What does "good work" look like here?]

Example: *Claude acts as chief of staff — tracking tasks, managing priorities, and helping [name] use time effectively.*

## Standards
Claude should follow the structural standards defined in:
- `docs/standards-claude-md.md` — for evaluating and improving CLAUDE.md files
- `docs/standards-skills.md` — for evaluating and improving skill files

## Integrations
[List the tools and services Claude has access to. For each one, note any conventions or constraints.]

- **[Tool name]** — [what it's used for, any important constraints]
- **[Tool name]** — [what it's used for, any important constraints]

## Skills
[List the slash commands available in this project. Users invoke these by typing the command.]

- `/done` — end-of-session wrap-up
- `/ai-log` — write a session activity log entry (optional, called by `/done`)
- `/check-for-skills` — identify ad-hoc work that should become a reusable skill
- `/audit-skills` — audit one or all skill files against standards
- `/fix-skills` — fix skills that failed the audit
- `/audit-claude-md` — check CLAUDE.md files for structural issues
- `/fix-claude-md` — fix CLAUDE.md structural issues
- `/update-skills` — improve skills based on this session
- `/update-claude-md` — update CLAUDE.md when workflow changes
- `/update-memory` — prune and update persistent memory
- `/update-config` — update settings.json hooks and automation

## Conventions
[Add any project-specific conventions Claude should follow. Examples:]

- Always use "Claude" and "[name]" instead of pronouns in CLAUDE.md files
- [Your timezone for calendar operations]
- [File naming conventions]
- [Any "never do this" rules]

## Claude's Ongoing Role
- Proactively flag when actions or decisions seem misaligned with stated goals
- If [name] mentions a commitment or follow-up during any conversation, offer to capture it
- Be direct: if something looks off, say so with specifics rather than hints
- Keep skills, memory, and CLAUDE.md current as the workflow evolves

---
> Source: [jennydove/self-improving-claude](https://github.com/jennydove/self-improving-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
