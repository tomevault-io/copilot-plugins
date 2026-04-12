## ai-template

> The project uses a custom ticketing system managed through npm scripts. Tickets are stored in the `tickets/` directory and organized into different states: `todo/`, `in-progress/`, `review/`, `done/`, `blocked/`, and `backlog/`.

# Ticketing System Rules

## Overview
The project uses a custom ticketing system managed through npm scripts. Tickets are stored in the `tickets/` directory and organized into different states: `todo/`, `in-progress/`, `review/`, `done/`, `blocked/`, and `backlog/`.

## Ticket States
- `todo/`: New tickets ready to be worked on
- `in-progress/`: Tickets currently being worked on
- `review/`: Tickets completed and awaiting review
- `done/`: Completed and approved tickets
- `blocked/`: Tickets blocked by dependencies or issues
- `backlog/`: Future tickets not yet ready for work

## Available Commands

### Create a New Ticket
```bash
npm run ticket create -t "Ticket Title" -d "Description" --type [feature|bug|infrastructure] -p [high|medium|low]
```
Options:
- `-t`: Ticket title (required)
- `-d`: Description (required)
- `--type`: Ticket type (feature, bug, or infrastructure)
- `-p`: Priority level (high, medium, or low)

Example:
```bash
npm run ticket create -t "Add user authentication" -d "Implement JWT-based auth" --type feature -p high
```

### List Tickets
```bash
npm run ticket list
```
This command shows all tickets in the system, organized by their current state.

### Move a Ticket
```bash
npm run ticket move TICKET-123 [todo|in-progress|review|done|blocked|backlog]
```
Example:
```bash
npm run ticket move INIT-001 in-progress
```

### Show Ticket Details
```bash
npm run ticket show TICKET-123
```
Example:
```bash
npm run ticket show INIT-001
```

## Ticket File Format
Each ticket is stored as a Markdown file with the following structure:
```markdown
# Ticket Title

## Description
Detailed description of the ticket

## Tasks
- [ ] Task 1
- [ ] Task 2

## Acceptance Criteria
- Criterion 1
- Criterion 2

## Priority: [High|Medium|Low]
## Type: [Feature|Bug|Infrastructure]
## Story Points: [number]
```

## Best Practices
1. Always create tickets with clear, actionable tasks
2. Include specific acceptance criteria
3. Use appropriate story points for estimation
4. Keep ticket descriptions concise but complete
5. Update ticket status promptly
6. Use the move command to track progress
7. Reference ticket numbers in commit messages and PRs

## Workflow
1. Create ticket in `todo/`
2. Move to `in-progress/` when starting work
3. Move to `review/` when ready for review
4. Move to `done/` after approval
5. Move to `blocked/` if dependencies prevent progress
6. Move to `backlog/` for future work

## Related Documentation
- [Development Guide](mdc:../docs/development.md)
- [Git Workflow](mdc:git-workflow.mdc)
- [Code Style Guide](mdc:code-style.mdc)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonstcyrx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
