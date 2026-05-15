## idiot-token

> Read this file before doing ANY work. No exceptions.

# Agent Operating Rules

Read this file before doing ANY work. No exceptions.

## Core Principles

1. Read before you work. Always read GLOBAL_AGENT_OPERATING_RULES.md, PROJECT_CONTROL.md, and TASK_LEDGER.md before starting any task.
2. No assumptions. Label all important facts: CONFIRMED | PROVISIONAL | UNVERIFIED | STALE | REJECTED.
3. Claim before you code. Mark a task IN_PROGRESS before starting. No two agents work the same task.
4. Branches only. Never commit directly to main/master. Use a short-lived feature branch and open a PR.
5. PR required for everything. Every change goes through a pull request with at least one review.
6. No destructive actions without owner approval. This includes: deleting files, running migrations, deploying to production, merging to main, restarting services, schema changes.
7. Update the ledger before stopping. Update TASK_LEDGER.md with status, files changed, blockers, and next actions.

## Required Reading Before Starting Work
- GLOBAL_AGENT_OPERATING_RULES.md
- PROJECT_CONTROL.md
- TASK_LEDGER.md

## Task Statuses
BACKLOG | READY | CLAIMED | IN_PROGRESS | BLOCKED | NEEDS_HUMAN | NEEDS_CONFIRMATION | REVIEW | DONE | FAILED | CANCELLED

## Requires Owner Approval
- Deleting files or folders
- Database migrations or schema changes
- Production deployments or server restarts
- Merging to main/master
- Auth, payment, billing, or customer data changes

## Branch & PR Workflow
1. Create a short-lived branch: feature/task-name or fix/issue-name
2. Make changes in that branch only
3. Open a PR with a clear description
4. Wait for review before merging
5. Delete the branch after merging

Source: GLOBAL_AGENT_OPERATING_RULES.md

---
> Source: [tiptophimp/idiot-token](https://github.com/tiptophimp/idiot-token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
