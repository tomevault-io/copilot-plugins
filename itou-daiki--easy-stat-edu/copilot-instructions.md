## easy-stat-edu

> エージェントオーケストレーション。複雑なタスク・設計決定・コードレビュー時に適用。


# Agent Orchestration

## Available Subagent Types

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| tdd-guide | Test-driven development | New features, bug fixes |
| code-reviewer | Code review | After writing code (MUST USE) |
| security-reviewer | Security analysis | Before commits, sensitive code |
| build-error-resolver | Fix build errors | When build fails |
| e2e-runner | E2E testing | Critical user flows |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation | Updating docs |

## Proactive Agent Usage (No Prompt Needed)

1. Complex feature requests → Use **planner** subagent
2. Code just written/modified → Use **code-reviewer** subagent
3. Bug fix or new feature → Use **tdd-guide** subagent
4. Architectural decision → Use **architect** subagent
5. Build fails → Use **build-error-resolver** subagent

## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:
- Launch multiple agents simultaneously when tasks are independent
- Maximum 4 concurrent agents
- Use TodoWrite to track multi-step tasks

## Performance Optimization

- Use `fast` model for quick, straightforward subagent tasks
- Reserve default model for complex reasoning tasks
- Keep context window healthy by scoping tasks tightly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itou-daiki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itou-daiki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
