## minecraft-mods-manager

> - **No Clarification Requests**: Do not ask for clarification unless critically blocked

# Governance: Communication Rules

## Core Communication Principles

### Autonomous Execution
- **No Clarification Requests**: Do not ask for clarification unless critically blocked
- **No Status Updates**: Do not say "proceeding," "on it," or similar phrases
- **Immediate Execution**: Execute the next step immediately without waiting
- **Self-Directed**: Work autonomously through task list without prompting
- **Complete Workflow**: Always finish the full issue lifecycle (create → implement → commit → push → close)

### Task Acknowledgment
When a new task is given:
- Acknowledge: "✅ Task added: [task]." (purpose: confirm receipt)
- Create GitHub issue: `gh issue create --title "Title" --body "Description" --label "type,priority"` (purpose: track detailed work)
- Update task list status (purpose: maintain high-level overview)
- Execute immediately (purpose: maintain momentum)
- Complete full workflow automatically (purpose: maintain development momentum)

### Ticket Update Communication
When updating tickets:
- Use Read-Ticket.ps1 to create backup files
- Modify the update file directly
- Use Update-Ticket.ps1 to apply changes
- Continue working immediately after update
- Report: "Updated ticket #[X] with [description]"

### Idle State Management
- **If no active task**: Prompt "What's next?"
- **If working on tasks**: Continue autonomously through task list
- **If blocked**: Clearly state the blocker and what's needed to proceed

## Automatic Workflow Communication

### MANDATORY: No Reminder Communication
- **Never ask** "should I commit this?" or "should I push?"
- **Never ask** "should I close the issue?" or "should I create an issue?"
- **Never wait** for user confirmation of standard workflow steps
- **Always execute** the complete workflow automatically

### Workflow Completion Communication
- **Report completion**: "✅ [Task] completed and issue #[X] closed"
- **Report next steps**: "Moving to next task: [task description]"
- **Report blockers**: "Blocked on [issue] - need [requirement]"

## Communication Patterns

### Issue Management Communication
- Use clear, actionable language in issue descriptions
- Reference related issues and pull requests
- Provide context for technical decisions
- Use appropriate labels and assignees

### Code Review Communication
- Be constructive and specific in feedback
- Reference relevant documentation and patterns
- Explain the reasoning behind suggestions
- Focus on code quality and maintainability

### Documentation Communication
- Write clear, concise documentation
- Use consistent terminology and formatting
- Include examples and use cases
- Keep documentation current with code changes

## Professional Standards

### Language and Tone
- Use professional, respectful language
- Be direct and concise
- Avoid jargon unless necessary
- Maintain positive, solution-focused approach

### Response Patterns
- Acknowledge requests promptly
- Provide clear, actionable responses
- Follow up on commitments
- Escalate issues when appropriate

### Collaboration Guidelines
- Respect team members' time and expertise
- Share knowledge and best practices
- Support continuous improvement
- Maintain open, transparent communication

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/survivorsunited)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/survivorsunited)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
