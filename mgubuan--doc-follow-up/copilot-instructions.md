## doc-follow-up

> Manage open documentation requests across your bookkeeping team: register what you need, track deadlines, draft contextual follow-up emails, and close the loop when documents arrive.

# Client Documentation Follow-Up

Manage open documentation requests across your bookkeeping team: register what you need, track deadlines, draft contextual follow-up emails, and close the loop when documents arrive.

## Folder Map

```
doc-follow-up/
├── CLAUDE.md              (you are here)
├── CONTEXT.md             (start here for task routing)
├── setup/
│   └── questionnaire.md   (onboarding -- run with "setup")
├── shared/
│   ├── tracker.csv        (master request tracker -- the backbone)
│   ├── follow-up-rules.md (deadline-driven follow-up schedule)
│   └── email-defaults.md  (firm name, signature, reply-to)
├── stages/
│   ├── 01-register/       (add new documentation requests)
│   ├── 02-assess/         (determine what needs follow-up today)
│   ├── 03-draft/          (write contextual follow-up emails)
│   ├── 04-receive/        (log incoming docs, run pre-check)
│   └── 05-review/         (accept, reject, or override)
└── test-data/             (LOTR-themed mock documents for testing)
```

## Triggers

| Keyword | Action |
|---------|--------|
| `setup` | Run onboarding questionnaire -- configures firm name, team, defaults |
| `status` | Show pipeline status and open item counts |
| `register` | Jump to Stage 01 to add a new request |
| `assess` | Jump to Stage 02 to check what needs follow-up today |
| `draft` | Jump to Stage 03 to write follow-up emails |
| `received` | Jump to Stage 04 to log an incoming document |
| `review` | Jump to Stage 05 to accept or reject a received document |

### How `status` works

Read `shared/tracker.csv`. Count items by status (Open, Received, Accepted, Returned). For Open items, calculate days to deadline and flag any that need follow-up today. Render:

```
Documentation Tracker: [firm-name]

Open: X items (Y need follow-up today)
Received (pending review): X items
Accepted (closed): X items
Returned (back to client): X items

Items needing follow-up today:
  - [Client] -- [Doc Type] -- [Description] -- due [date]
```

## Routing

| Task | Go To |
|------|-------|
| Add a new documentation request | `stages/01-register/CONTEXT.md` |
| Check what needs follow-up today | `stages/02-assess/CONTEXT.md` |
| Draft follow-up emails | `stages/03-draft/CONTEXT.md` |
| Log a received document | `stages/04-receive/CONTEXT.md` |
| Review and accept/reject a document | `stages/05-review/CONTEXT.md` |

## What to Load

| Task | Load These | Do NOT Load |
|------|-----------|-------------|
| Register a request | `shared/tracker.csv`, `stages/01-register/references/*` | Stages 02-05 |
| Assess open items | `shared/tracker.csv`, `shared/follow-up-rules.md` | Stages 01, 03-05, references |
| Draft follow-ups | `stages/02-assess/output/*`, `shared/tracker.csv`, `shared/email-defaults.md`, `stages/03-draft/references/*` | Stages 01, 04-05 |
| Log received doc | `shared/tracker.csv`, `stages/04-receive/references/*` | Stages 01-03, 05 |
| Review received doc | `stages/04-receive/output/*`, `shared/tracker.csv`, `stages/05-review/references/*` | Stages 01-03 |

## Stage Handoffs

This is NOT a linear pipeline. Stages can be run in any order based on what the team needs right now. The tracker CSV is the shared state that connects everything. Every stage reads from it and some stages update it.

- Stage 01 WRITES new rows to the tracker
- Stage 02 READS the tracker to find items due for follow-up
- Stage 03 READS Stage 02 output + tracker to draft emails
- Stage 04 UPDATES tracker status from Open to Received
- Stage 05 UPDATES tracker status to Accepted or Returned (back to Open)

---
> Source: [mgubuan/doc-follow-up](https://github.com/mgubuan/doc-follow-up) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
