## 801-feature-workflow

> FOLLOW feature development workflow WHEN implementing new features TO ensure consistent, tested, and documented changes


# Feature Development Workflow

## Context
- When implementing new features from the features documentation
- When making significant changes that require testing and documentation
- When working on a feature branch

## Requirements
- Start from up-to-date main branch
- Create feature branch with descriptive name (feature/feature-name)
- Locate and review feature documentation in docs/features directory
- Follow documentation requirements systematically
- IMPORTANT Commit frequently to trigger pre-commit hooks (linting, formatting, tests)
- CRITICAL Always stage ALL modified files after making changes
- CRITICAL Always push changes to remote after committing
- Add comprehensive tests for new functionality
- Push completed work to remote for PR review

## Examples
<example>
# Good workflow
1. git checkout main && git pull
2. git checkout -b feature/server-info
3. Review docs/features/002-server-info-resource.md
4. Implement ServerInfo interface
5. Add getServerInfo method with tests
6. git add . # Stage ALL changes
7. Commit to run hooks: "feat: add server info interface"
8. git push origin feature/server-info # Push changes immediately
9. Implement ServerStatus interface
10. Add getServerStatus method with tests
11. git add . # Stage ALL changes again
12. Commit to run hooks: "feat: add server status resource"
13. git push origin feature/server-info # Push changes again
14. Fix any linting issues
15. git add . && git commit -m "fix: linting issues"
16. git push origin feature/server-info
</example>

<example type="invalid">
# Poor workflow
1. Start coding without checking docs
2. Make all changes in one big commit
3. Skip tests or add them later
4. Leave changes unstaged
5. Forget to push changes to remote
6. Push directly to main
7. Fix linting issues after PR
</example>

## Critical Points
<critical>
- ALWAYS work from feature documentation
- NEVER skip tests for new functionality
- Commit OFTEN to utilize pre-commit hooks
- ALWAYS stage ALL modified files after making changes
- ALWAYS push changes to remote after committing
- Keep commits focused and well-described
</critical> 

---
> Source: [StuMason/coolify-mcp](https://github.com/StuMason/coolify-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
