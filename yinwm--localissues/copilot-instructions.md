## local-issues

> When agent works with user story , issues or other requirments and task jobs

# Local Issues Management System

## Directory Structure

- `/.issues` - Root directory for all issues
  - First level: Milestone folders (e.g., `m001-InitProject`)
  - Second level: Issue files within each milestone
  - `/.issues/templates` - Issue templates, for story and issue
    - @story.md - For user stories
    - @issue.md - For features, fixes, and tests

## Issue Types and Naming

Issue files follow the naming pattern: `{id}-{type}-{description}.md`
- `{id}`: Globally unique identifier across all issues, cross all issues in any sub-directory of /issues/*
- `{type}`: One of the following:
  - `story` - User story describing complete user value
  - `feat` - New feature implementation
  - `fix` - Bug fix
  - `test` - Test implementation
- `{description}`: Brief kebab-case description

## Issue States

Tasks and issues can have the following states:
- `[ ]` - Not started
- `[x]` - Completed
- `[-]` - In progress
- `[*]` - Skipped
- `[!]` - Abandoned

## Templates

Two types of issue templates are available in `/.issues/templates/`:
1. @story.md : For user stories
2. @issue.md : For features, fixes, and tests

## File Modification Rules
- When creating 
    - New issue, use @issue.md template
    - New story, use @story.md template
    - Always check all issues and stories in /.issues to make sure the id is unique
- Only modify task/issue states in existing files
- Do not alter other content without creating a new issue
- Always update Status History when changing states 

---
> Source: [yinwm/LocalIssues](https://github.com/yinwm/LocalIssues) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
