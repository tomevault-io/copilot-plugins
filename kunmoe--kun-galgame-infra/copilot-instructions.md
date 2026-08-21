## grok-executor

> Orchestrator/executor split - this agent drafts task books and dispatches them to the local `grok` CLI as executor; only critical, error-prone steps are executed directly


# Grok-as-executor dispatch mode

The agent in this session is the **orchestrator** (task-book author + acceptor). Implementation and investigation work is dispatched to the locally installed `grok` CLI (Grok Build) as the **executor subagent**. Do NOT use Cursor's built-in Task subagents for execution work; use `grok`.

## How to dispatch

1. Write a self-contained task book to `/tmp/<task-slug>.md`, then run from the repo root, in the background:

```bash
grok --prompt-file /tmp/<task-slug>.md --output-format plain --max-turns 200 \
  > /tmp/<task-slug>-report.md 2>/tmp/<task-slug>-stderr.log
```

2. Headless default permissions allow file reads and read-only shell without approval; write control rides on the task book's discipline section plus your post-run checks. For risky tasks add `--permission-mode` / `--sandbox`.
3. Never run two grok dispatches concurrently on overlapping paths — one writer per path. Sequential dispatches only.

## Task book requirements (what makes executors perform well)

- **Write it in English** — executors follow English task books more reliably.
- Self-contained: the executor sees nothing of this conversation. State repo path, branch, HEAD, where the code lives, and every relevant prior decision inline.
- Explicit scope AND out-of-scope lists; named acceptance criteria (test names, commands with expected output); a required report structure.
- All design adjudications are made by the orchestrator in the task book — never delegate an open design decision to the executor. If mechanics genuinely depend on code details, state the invariant to enforce plus the precedent to follow, and require the executor to report the chosen mechanics for acceptance.
- A discipline section: writable paths (exact), forbidden operations (git add/commit/checkout/reset/stash, DB access unless a DSN is explicitly provided, starting services), and "report, don't work around" for anything unexpected.

## What the orchestrator never delegates

- Task-book adjudications and scope calls.
- Git commits (explicit paths only), pushes, PRs, and anything touching shared git state.
- Database provisioning (`scripts/ephemeral-test-db.sh create <slug>`; pass the DSN into the grok dispatch env; `drop` it when done) and all migrations.
- Cross-repo docs sync (`../kungal-docs` `docs:sync --write` + `docs:audit`) and production operations.
- Final acceptance: after every dispatch run `git status --porcelain` (only expected paths changed), spot-check the report's highest-stakes claims against the code, and independently re-run the tests/gates for write tasks. Trust the report's structure, verify its conclusions.

---
> Source: [KunMoe/kun-galgame-infra](https://github.com/KunMoe/kun-galgame-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
