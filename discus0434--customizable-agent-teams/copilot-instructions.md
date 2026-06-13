## customizable-agent-teams

> - `TEAM_AGENT_WORKTREE`

# Agent Team Rules

## Identify

```bash
make team-identity
```

Primary identity:

- `TEAM_AGENT_ID`
- `TEAM_AGENT_ROLE`
- `TEAM_AGENT_CLI`
- `TEAM_AGENT_MODEL`
- `TEAM_AGENT_WORKTREE`
- `TEAM_SESSION`

Read before task work:

1. `.agents/docs/TEAM_PROTOCOL.md`
2. `.agents/docs/MEMORY.md`
3. `$TEAM_ROOT/.agents/queue/tasks/<task_id>.md`

Queue, inbox, report, review, and integration artifacts live under `TEAM_ROOT`. In a worker worktree, use the Make targets or absolute paths from messages instead of treating the local `.agents/queue/` directory as canonical.

## Tooling

- Use `gh` for GitHub operations such as PR creation, PR status, issue comments, review comments, and CI inspection.
- When repository environment variables must be loaded, run commands through `direnv exec . <command>`.
- Missing required tools are blockers. Report the missing command.

## Verification

- `make post-change` is the required change gate.
- `make smoke` confirms the user-visible behavior selected during bootstrap.

## User Interface

- Human users give project instructions directly in the lead tmux pane.
- Use file mailbox plus tmux nudges for agent-to-agent messages, including lead-to-worker dispatch and worker-to-lead questions.
- When a pane receives `inbox <agent_id>`, run `make inbox AGENT=<agent_id>` and process the unread message body from `TEAM_ROOT`.
- If a pane shows an unsubmitted `inbox <agent_id>` prompt, submit it with `make team-submit AGENT=<agent_id>`.

## Roles

### lead

- Do small, single-agent changes directly.
- Create task files for delegated work.
- Dispatch tasks with `make team-send`.
- Decide worker questions after review.
- Integrate only `done` reports with `OK` reviews.
- Edit `.agents/docs/MEMORY.md` after reviewing memory proposals.

Direct work is allowed when ownership is clear, the change is small, and lead can run the relevant task-specific checks plus:

```bash
make post-change
make smoke
```

Use workers when isolation, parallelism, review follow-up, or separate ownership is useful.

### worker

- Read inbox and task file.
- Claim the task before changing files.
- Work only on the task branch checked out by `make claim`.
- Respect `Allowed paths` and `Do not modify`.
- Run task-specific verification, `make post-change`, and `make smoke`.
- Commit the finished task branch before review.
- Fill the report with summary, changed files, verification commands, results, and evidence before review.
- Run noninteractive review and handle the result.
- Report blockers, questions, verification gaps, and memory proposals.
- Submit memory changes as proposals. Lead edits `.agents/docs/MEMORY.md`.

Direct lightweight requests without a task file, such as `TYPE=retro` or `TYPE=note`, do not use claim, commit, review, or integration. Follow the inbox body, write the requested artifact, and mark the message processed.

### verifier

- Runs only through `make review`.
- Reviews task, report, committed diff, and verification evidence.
- Returns `Decision: OK`, `Decision: FIX`, or `Decision: ASK_LEAD`.
- Does not edit files or own a worktree.

## Worker Flow

```bash
make claim TASK=<task_id> AGENT=<agent_id>
```

`make claim` checks `Owner`, `Branch`, clean worker worktree, claim state, and checks out:

```text
task/<agent_id>/<task_id>
```

After implementation:

```bash
make post-change
make smoke
git add <changed-files>
git commit -m "<task_id>: <summary>"
make report TASK=<task_id> AGENT=<agent_id> STATUS=needs-review
# Edit .agents/queue/reports/<task_id>_<agent_id>.md with concrete verification evidence.
make review TASK=<task_id> AGENT=<agent_id>
```

Handle review:

- `OK`: update the report to `done`.
- `FIX`: fix, rerun checks, commit, report `needs-review`, and review again.
- `ASK_LEAD`: write the question in the report and notify lead.

Finish:

```bash
make report TASK=<task_id> AGENT=<agent_id> STATUS=done
make inbox AGENT=<agent_id>
make inbox AGENT=<agent_id> MARK=<message_id>
```

After review, recheck inbox and mark any review notification already handled through the review artifact.

## Lead Integration

Check status:

```bash
make team-status
```

Integrate only tasks shown as `ready-to-integrate`:

```bash
make integrate TASK=<task_id> AGENT=<agent_id>
```

Integration requires:

- report `Status: done`
- review `Decision: OK`
- clean worker worktree
- clean lead worktree
- unchanged worker head since report

`make integrate` performs a `--no-ff` merge, runs `make post-change` and `make smoke`, and writes `.agents/queue/integrations/<task_id>_<agent_id>.md`.

## Memory

- Lead is the only editor of `.agents/docs/MEMORY.md`.
- Workers write durable lessons to `.agents/queue/memory_proposals/`.
- Store only lessons that will change future agent behavior.

---
> Source: [discus0434/customizable-agent-teams](https://github.com/discus0434/customizable-agent-teams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
