## claude-agents

> Detailed documentation for the seven Claude Code development agents and the Athena PR Reviewer skill.

# Agent Documentation

Detailed documentation for the seven Claude Code development agents and the Athena PR Reviewer skill.

## Atlas Jira Analyst

### Purpose
Atlas carries the weight of project knowledge, extracting and analyzing Jira tickets, epics, and related stories. It compiles comprehensive requirements documentation to ensure developers have full context before starting work.

### Trigger Phrases (Proactive)
The agent automatically activates when you mention:
- Jira issue IDs (e.g., "PROJ-1234")
- "What should I implement for..."
- "Get the Jira details"
- "What's the context for this feature"
- Working on a feature branch with a Jira ID

### Capabilities
- Extracts issue ID from git branch names
- Retrieves issue description, title, and acceptance criteria
- Collects all comments with special attention to decisions and clarifications
- Fetches parent epic context and related stories
- Gathers Definition of Done
- Links to Confluence documentation

### Input Formats
```bash
# Explicit issue ID
"Get context for PROJ-1234"

# From current branch
"What's the Jira context for this branch?"

# General request
"I need to understand what to implement"
```

### Output Format
```
# JIRA ISSUE: [ISSUE-ID] - [Title]

## SUMMARY
[Clear description of what needs to be done]

## METADATA
Type: Story
Status: In Progress
Priority: High
...

## ACCEPTANCE CRITERIA
1. [Criterion 1]
2. [Criterion 2]

## KEY DECISIONS AND CLARIFICATIONS
- [@user]: "Important clarification"
...
```

### Example Usage
```
User: "I need to understand what I should implement for PROJ-567"
Assistant: [Invokes atlas-jira-analyst to gather full Jira context]
```

---

## Heimdall PR Guardian

### Purpose
Heimdall stands watch at the bridge to merge, monitoring pull request status including comments, CI/CD checks, approvals, and merge blockers. Returns raw status data without analysis.

### Trigger Phrases (Proactive)
The agent automatically activates when you mention:
- "PR status"
- "PR comments" / "check comments" / "review comments"
- "What's blocking my PR"
- "PR approvals"
- "Merge readiness"
- "PR feedback"

### Capabilities
- Detects PR from current branch or accepts PR number/URL
- Fetches all types of comments (general, review, code-specific)
- Extracts comment IDs for responding/resolving
- Monitors CI/CD check status with failure logs
- Tracks approvals and review requests
- Identifies merge blockers

### Input Formats
```bash
# PR number
"Check status of PR #123"

# Full URL
"Get comments for https://github.com/org/repo/pull/456"

# Current branch
"What's blocking my PR?"
```

### Output Format
```
# PULL REQUEST STATUS: #123 - [Title]

## COMMENTS STATUS
Total Comments: 10 (3 resolved, 7 unresolved)

### UNRESOLVED COMMENTS REQUIRING ACTION:
1. @reviewer (2 days ago): "Comment text"
   - Comment ID: 1234567890
   - Review ID: 9876543210
   - File: src/app.js:42
   - Has replies: Yes (2)
   
## CI/CD STATUS
Required Checks: 3 of 5 passing

### FAILING CHECKS:
- build-test: FAILED
  [Error logs]

## ACTION REQUIRED
1. Respond to 4 comments
2. Fix 2 failing checks
...
```

### Example Usage
```
User: "Check the comments on my PR"
Assistant: [Invokes heimdall-pr-guardian to gather PR status and comments]
```

---

## Hermes PR Courier

### Purpose
Hermes swiftly delivers comprehensive information about pull requests, collecting metadata, file changes, commit history, and linked issues without adding interpretation or opinions.

### Trigger Phrases (Proactive)
The agent automatically activates when you mention:
- "What's in PR #..."
- "Get PR details"
- "Show me the files changed"
- "PR changes"
- Reviewing PRs
- Documenting changes for release notes

### Capabilities
- Fetches PR metadata (title, description, author, timestamps)
- Collects file changes with addition/deletion counts
- Categorizes files by type (frontend/backend/tests/docs/etc.)
- Analyzes commit history and types
- Links to related issues
- Calculates PR size (XS/S/M/L/XL)

### Input Formats
```bash
# PR number
"What's in PR #789"

# Full URL
"Get details for https://github.com/org/repo/pull/321"

# With repository context
"Show me org/repo#456"
```

### Output Format
```
# PULL REQUEST: #1234 - [Title]

## METADATA
Author: @username
State: OPEN
Branch: feature → main
Labels: [bug, high-priority]

## CHANGE SUMMARY
Total Changes: 570 lines (+450, -120) across 15 files
PR Size: MEDIUM

## FILES CHANGED BY CATEGORY

### Frontend (5 files, +200 -50 lines)
- src/components/Button.tsx (+45, -10)
...

## COMMITS (5 total)
1. feat: add Button component
   - SHA: abc123d
   - Type: feature

## LINKED ISSUES
Closes: #456 - "Issue title"
```

### Example Usage
```
User: "What's in PR #1234?"
Assistant: [Invokes hermes-pr-courier to gather PR content information]
```

---

## Minerva Notion Oracle

### Purpose
Minerva accesses the collective wisdom stored in Notion workspaces, retrieving documentation, meeting notes, project information, and any other knowledge stored in your Notion pages.

### Trigger Phrases (Proactive)
The agent automatically activates when you mention:
- "Find documentation about..."
- "Check Notion for..."
- "Look up in our docs"
- "Search the knowledge base"
- "Get meeting notes"
- Notion page URLs

### Capabilities
- Searches across Notion workspaces
- Retrieves specific pages by URL or ID
- Fetches page content in markdown format
- Searches by keywords and filters
- Accesses databases and their content
- Follows page hierarchies

### Input Formats
```bash
# Search query
"Find our API authentication documentation"

# Direct page URL
"Get https://notion.so/workspace/Page-Title-123abc"

# Topic search
"Search Notion for deployment process"

# Meeting notes
"Get the architecture review meeting notes"
```

### Output Format
```
# NOTION SEARCH RESULTS

## FOUND PAGES (3 matches)

### 1. API Authentication Guide
- **Page ID**: 123abc456def
- **Last Updated**: 2024-01-15
- **Space**: Engineering Docs
- **URL**: https://notion.so/workspace/API-Auth-123abc

**Content Preview:**
[First 500 characters of page content]

### 2. Authentication Best Practices
- **Page ID**: 789ghi012jkl
- **Last Updated**: 2024-01-10
- **Space**: Security Guidelines

## FULL CONTENT: API Authentication Guide

[Complete markdown content of the most relevant page]
```

### Example Usage
```
User: "Find our Redis caching documentation"
Assistant: [Invokes minerva-notion-oracle to search and retrieve Redis-related documentation]
```

---

## Customization

### Modifying Trigger Phrases
Edit the `description` field in each agent's frontmatter to add or modify trigger phrases. Include "PROACTIVELY USED" followed by the phrases.

### Changing Output Format
Modify the output format section in each agent's prompt. Keep it structured and LLM-optimized for best results.

### Switching Models
Change `model: sonnet` to `model: opus` in the frontmatter for more complex reasoning (higher cost).

### Adding Tools
Add tool names to the `tools:` list in the frontmatter. Ensure the tools are available in your Claude Code environment.

## Tips for Effective Usage

1. **Let agents work proactively**: Don't explicitly ask to use agents; just mention the keywords
2. **Use parallel execution**: Multiple agents can run simultaneously for different tasks
3. **Chain agent outputs**: Use one agent's output as context for another
4. **Keep agents updated**: Pull latest changes and re-run installer for updates

## Common Patterns

### Starting New Feature Work
```
"I'm starting work on PROJ-123, check if there's a PR already"
→ Triggers both atlas-jira-analyst and heimdall-pr-guardian
```

### Comprehensive PR Review
```
"Review PR #456"
→ Triggers athena-pr-reviewer (which orchestrates other agents)
```

### Code Review with Context
```
"Review PR #456 and check our coding standards in Notion"
→ Triggers athena-pr-reviewer and minerva-notion-oracle
```

### Checking PR Readiness
```
"Is my PR ready to merge? Check comments and status"
→ Triggers heimdall-pr-guardian
```

### Documentation Lookup
```
"Find our deployment process documentation"
→ Triggers minerva-notion-oracle
```

---

## Athena PR Reviewer (Skill)

### Purpose
Athena brings wisdom to code reviews through a multi-LLM approach. As a **skill** (not an agent), she orchestrates 8 parallel reviewers to provide comprehensive, confidence-scored feedback.

### Trigger Phrases (Proactive)
The skill automatically activates when you mention:
- "Review PR #..."
- "Review this pull request"
- "PR review"
- "Code review"
- "Review CSD-123" (Jira ticket)

### Architecture
Athena runs **8 reviewers in parallel**:

**External LLMs (via bash script):**
- **Gemini**: General code review with large context
- **Codex**: General code review from OpenAI

**Claude Specialists (via Task tool):**
1. **Comment Analyzer**: Documentation quality, technical debt markers
2. **Test Analyzer**: Test coverage gaps, missing edge cases
3. **Error Hunter**: Silent failures, missing error handling
4. **Type Reviewer**: Type design quality, type safety
5. **Code Reviewer**: General quality, bugs, security (uses Opus)
6. **Simplifier**: Unnecessary complexity, over-engineering

### Confidence Scoring
Each finding includes a confidence score (0-100):
- **90-100**: Certain - clear violation, verifiable
- **70-89**: Likely - probable issue, needs verification
- **50-69**: Possible - might be intentional
- **<50**: Not reported (too speculative)

### Consensus Boosting
Items flagged by 2+ reviewers get priority bumped:
| Reviewers | Original | Final |
|-----------|----------|-------|
| 3+ | High | Critical |
| 2 | High | Critical |
| 3+ | Medium | High |
| 2 | Medium | High |

### Data Gathering
The skill gathers comprehensive context via `gather-context.sh`:
- PR metadata and diff
- Jira ticket and epic context
- CLAUDE.md guidelines from repo
- Git blame for changed files
- Prior PR comments on same files

### Output Format
```markdown
# PR Review: {Title} (#{Number})

## Requirements Status
| Requirement | Status | Notes |
|-------------|--------|-------|

## Action Items

### Critical (consensus)
- [ ] file:line - issue - fix [Gemini + Codex + Claude-errors] (3+, avg 92%)

### High Priority
- [ ] file:line - issue - fix [Gemini + Claude-tests] ← boosted (2, avg 85%)

### Medium Priority
- [ ] file:line - issue - fix [Claude-simplify] (88%)

### Suggestions
- improvements

## Recommendation: APPROVE / REQUEST_CHANGES
```

### Example Usage
```
User: "Review PR #456"
Assistant: [Invokes athena-pr-reviewer skill which runs 8 parallel reviews]
```

---
> Source: [emiperez95/claude-agents](https://github.com/emiperez95/claude-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
