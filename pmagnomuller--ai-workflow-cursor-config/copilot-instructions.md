## ai-workflow-cursor-config

> When the user issues workflow commands, follow these patterns:

# Cursor IDE Rules for AI-Assisted Development Workflow

## Workflow Commands

When the user issues workflow commands, follow these patterns:

### "Start task [Jira link]"
1. Fetch the Jira ticket details using MCP
2. Extract key information: title, description, acceptance criteria, labels
3. List relevant files in the codebase that might need changes
4. Suggest initial implementation steps
5. Create a feature branch if needed (format: `feature/TICKET-NUMBER-short-description`)
6. Set up the initial commit structure

### "Update documentation"
1. Identify what documentation needs updating based on code changes
2. Use Notion MCP to find relevant documentation pages
3. Update documentation with code changes, examples, and context
4. Ensure documentation stays in sync with code

### "Open PR"
1. Review all changes in the current branch
2. Generate a comprehensive PR description including:
   - Summary of changes
   - Related Jira ticket link
   - Testing notes
   - Screenshots if applicable
3. Use GitHub MCP to create the pull request
4. Add appropriate labels and reviewers
5. Link the PR to the Jira ticket

### PR Review Assistance
When reviewing PRs:
1. Analyze code changes for potential issues
2. Summarize changes in plain language
3. Suggest improvements or optimizations
4. Check for consistency with codebase patterns
5. Verify documentation is updated

## Code Style Guidelines
- Follow existing code patterns in the repository
- Write clear, self-documenting code
- Add comments for complex logic
- Maintain consistency with team conventions

## Context Awareness
- Always consider the full context: Jira ticket, related PRs, documentation
- Cross-reference information across tools (Jira, GitHub, Notion)
- Maintain awareness of the broader project goals

## Error Handling
- Provide clear error messages
- Suggest solutions when issues arise
- Log important actions for debugging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pmagnomuller) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
