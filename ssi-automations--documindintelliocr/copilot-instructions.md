## documindintelliocr

> **MANDATORY: When an issue is resolved and successfully completed, you MUST automatically close the GitHub issue using the GitHub MCP tools.**


## GitHub Issue Management Rule

**MANDATORY: When an issue is resolved and successfully completed, you MUST automatically close the GitHub issue using the GitHub MCP tools.**

### Rule Details:

- **Trigger**: Whenever you complete work that resolves a GitHub issue (whether explicitly mentioned by issue number or implicitly referenced)
- **Required Action**: Use `mcp_github_update_issue` to set the issue state to "closed"
- **No Exceptions**: This rule applies to ALL GitHub issues, regardless of repository or complexity

### Implementation Steps:

1. **Identify the Issue**: If an issue number is mentioned, use it directly. If work relates to an issue but no number is given, search for the relevant issue first using `mcp_github_search_issues` or `mcp_github_list_issues`
2. **Get Issue Details**: Use `mcp_github_get_issue` to confirm the issue exists and is currently open
3. **Close the Issue**: Use `mcp_github_update_issue` with `state: "closed"` parameter
4. **Confirm Completion**: Verify the issue was successfully closed and inform the user

### Required Parameters for Closing:

- `owner`: Repository owner (get from user context or GitHub API)
- `repo`: Repository name (get from user context or GitHub API)
- `issue_number`: The issue number to close
- `state`: Must be set to "closed"

### Example Scenarios:

- ✅ "I fixed the login bug mentioned in issue #5" → Close issue #5
- ✅ "The password strength meter is now implemented" → Search for related issue, then close it
- ✅ "Completed the user authentication feature" → Find corresponding issue and close it

### Error Handling:

- If issue doesn't exist, inform user and ask for clarification
- If issue is already closed, acknowledge and continue
- If API call fails, retry once before reporting error

**This rule ensures proper GitHub project management and maintains accurate issue tracking.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SSI-Automations) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
