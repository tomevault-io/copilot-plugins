## lfk

> You are Claude Code. I use specialized agents and skills for complex tasks.

# CLAUDE.md

## Core Philosophy (MANDATORY)

You are Claude Code. I use specialized agents and skills for complex tasks.

**Philosophy**: Agent-first design, parallel execution, plan before action, test before code, security always.

**Key Principles:**
1. **Agent-First**: Delegate to specialized agents for complex work
2. **Parallel Execution**: Use Task tool with multiple agents when possible
3. **Plan Before Execute**: Use Plan Mode for complex operations
4. **Test-Driven**: Write tests before implementation
5. **Security-First**: Never compromise on security

## General Rules

- When possible, always use sub-agent driven development.
- If you implement a new feature or update an existing one, make sure to update the documentation in the readme file and any relevant documentation files or the in-app help.
- If you change any hotkey bindings, make sure to update the documentation and inform the team.
- Always delegate to specialized agents.
- Always run security check agents.
- Always create unit, functional and e2e tests where applicable.
- Use research organizations and compare their solution.
- Always use agentic mode for non-trivial tasks.
- Always search for refactor opportunities and reusable functions before implementing a new feature or fixing a bug.
- Wherever possible use red/green TDD.
- Research and follow best practices for code quality, security and performance.
- Ask questions if you are unsure about any requirements or implementation details.
- Whenever it's possible, prepare Makefile tasks to automate common development tasks like testing, linting, formatting, building and deploying. Make sure the Makefile is well documented and easy to use, includes help messages, help target and clear instructions for each task.

### Privacy

- Always redact logs; never paste secrets (API keys/tokens/passwords/JWTs)
- Review output before sharing - remove any sensitive data

### Code Style

- No emojis in code, comments, or documentation
- Prefer immutability - never mutate objects or arrays
- Many small files over few large files
- 200-400 lines typical, 800 max per file

### Git

- Wherever possible, create git hooks to automate testing, linting and formatting before the code is committed.
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- Always test locally before committing
- Small, focused commits

### Testing

- TDD: Write tests first
- 80% minimum coverage
- Unit + integration + E2E for critical flows

### Knowledge Capture

- Personal debugging notes, preferences, and temporary context → auto memory
- Team/project knowledge (architecture decisions, API changes, implementation runbooks) → follow the project's existing docs structure
- If the current task already produces the relevant docs, comments, or examples, do not duplicate the same knowledge elsewhere
- If there is no obvious project doc location, ask before creating a new top-level doc

---

## Success Metrics

You are successful when:
- All tests pass (80%+ coverage)
- No security vulnerabilities
- Code is readable and maintainable
- User requirements are met

---
> Source: [janosmiko/lfk](https://github.com/janosmiko/lfk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
