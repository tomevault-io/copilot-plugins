## agile-team-skill

> Drop this into any codebase. You get 7 specialist agents, collaborative agile ceremonies,

# Agile Team — AI-Powered Scrum for Any Project

Drop this into any codebase. You get 7 specialist agents, collaborative agile ceremonies,
and persistent team memory. No lock-in. Works with any language or framework.

---

## What's Here

```
.claude/
├── agents/                    ← 7 specialist agents
│   ├── po-agent.md            Product Owner
│   ├── pm-agent.md            Scrum Master
│   ├── dev-agent.md           Developer
│   ├── qa-agent.md            QA Engineer
│   ├── pr-reviewer-agent.md   PR Reviewer
│   ├── security-analyst-agent.md  Security Analyst
│   └── tech-lead-agent.md     Tech Lead
└── commands/                  ← 30 slash commands

memory/                        ← Persistent team state
├── NEXT.md                    Exact next action (session continuity)
├── STATE.md                   Current sprint
├── BACKLOG.md                 Product backlog (## Index at top — read index first)
├── ARCHIVE.md                 Completed stories (append-only — moved here by /complete)
├── DECISIONS.md               Architecture decisions
└── LEARNINGS.md               Team learnings (append-only)
```

**Token discipline:** ceremonies read the BACKLOG.md `## Index` section first and extract a
single story body (`awk '/^- \[.\] STORY-XXX:/,/^---$/'`) only when acting on it. In chains,
the orchestrator extracts once and passes the story body in each agent's prompt — agents do
not re-read BACKLOG.md. ARCHIVE.md is never read during ceremonies.

---

## The Collaboration Principle: Nesting = Collaboration

No agent works alone. Every ceremony runs a **collaboration chain** — agents give their
perspective in sequence, then one agent synthesizes into a shared artifact.

```
/review chain:
  qa           ──→  quality gate (tests + AC) — STOP if fail, no code review of broken code
  pr-reviewer  ──→  code quality findings
  security     ──→  vulnerability findings
  tech-lead    ──→  architecture findings
  po           ──→  [SYNTHESIZES ALL] → APPROVED / CHANGES REQUESTED + BACKLOG items
```

The PO is the hub. Issues that don't block merge go straight to BACKLOG.md. Nothing is lost.

---

## The Dev Pipeline

Every story follows this exact flow:

```
/sprint-plan  po proposes → dev commits capacity → tech-lead estimates → qa validates AC → pm finalizes
      ↓
/standup      daily: done / doing / blocked — blockers get owner + mitigation
      ↓
/new-task     po selects story → tech-lead specs → dev confirms and starts
      ↓
[implement]
      ↓
/review       qa gate first → code review → security → tech-lead → po verdict
              APPROVED → /complete | CHANGES REQUESTED → fix → /review again
      ↓
/complete     commit + close story → /new-task (more stories) or /sprint-close (done)
      ↓
/sprint-close → /retro → /sprint-plan (next sprint)
```

If blocked at any point: `/unblock STORY-XXX "what resolved it"`

---

## The 7 Agents

| Agent | Role | Owns | Hard Veto |
|---|---|---|---|
| `po-agent` | Product Owner | BACKLOG.md, sprint goal, user stories | No |
| `pm-agent` | Scrum Master | STATE.md, NEXT.md, ceremonies | No |
| `dev-agent` | Developer | Code, implementation, capacity estimates | No |
| `qa-agent` | QA Engineer | Test strategy, acceptance criteria | YES — no ship without tests |
| `pr-reviewer-agent` | PR Reviewer | Code review, merge gate | Soft — can block PR |
| `security-analyst-agent` | Security | Vulnerability scan, risk register | Soft — can block PR |
| `tech-lead-agent` | Tech Lead | DECISIONS.md, architecture, estimates | No |

---

## Ceremony Map

| Command | Collaboration Chain | Output Artifact |
|---|---|---|
| `/standup` | dev → qa → security → tech-lead → pm synthesizes → po notes | STATE.md updated |
| `/sprint-plan` | po proposes → dev estimates capacity → tech-lead complexity → qa validates AC → security flags → pm finalizes | Sprint in STATE.md |
| `/sprint-close` | pm reads velocity → po reviews stories → all agents sign off | STATE.md CLOSED |
| `/retro` | all agents reflect → pm facilitates → po backlogs actions → learnings logged | LEARNINGS.md |
| `/review` | qa gate → pr-reviewer → security → tech-lead → po synthesizes | Verdict + BACKLOG.md |
| `/stories` | po writes → qa adds test scenarios → security adds constraints → tech-lead adds notes | BACKLOG.md entry |
| `/backlog` | po leads → tech-lead estimates → qa validates AC → security flags risk | BACKLOG.md prioritized |
| `/new-task` | po selects → tech-lead specs → pm assigns → dev confirms | IN_PROGRESS in STATE.md |
| `/status` | pm reads state → all agents report health | Full project picture |
| `/unblock` | tech-lead confirms resolution → pm clears STATE.md → NEXT.md updated | Blocker removed |

---

## Session Protocol

**Start of session:**
```
cat memory/NEXT.md     # exact pickup point — always start here
cat memory/STATE.md    # sprint status
```

**Then:**
- Start of day → `/standup`
- Need next work → `/new-task`
- Story done → `/review` then `/complete STORY-XXX`
- Blocked → `/unblock STORY-XXX`
- End of sprint → `/sprint-close` then `/retro`

**End of session:**
- Always overwrite `memory/NEXT.md` with the exact next action
- One commit per completed story: `feat(area): description — closes STORY-XXX`

---

## Security Hook Configuration

### Pre-commit secret scanning (gitleaks)

The installer adds a pre-commit git hook that runs `gitleaks protect --staged`
before every commit. If a secret pattern is detected, the commit is blocked.

**False positive suppression:** Add `# gitleaks:allow` as an inline comment on
the offending line:

```python
TEST_API_KEY = "sk-test-placeholder"  # gitleaks:allow
```

**Emergency bypass** (use sparingly, document why in commit message):
```bash
git commit --no-verify -m "reason: ..."
```

**Not installed?** The hook exits 0 with a warning — commits still work.
Install gitleaks to activate: `brew install gitleaks`

---

### Secret scan override (Write/Edit tool)
The `pre-tool-use.sh` hook scans every Write and Edit tool call for known secret
patterns (OpenAI keys, GitHub tokens, AWS keys, Bearer tokens, PEM private keys).

If you hit a false positive on a legitimate code constant (e.g. a test fixture
with a placeholder key pattern), you can bypass the scan for a single session:

```bash
export SKIP_SECRET_SCAN=1
```

**Rules:**
- Document the reason in your commit message when bypassing
- Unset after use: `unset SKIP_SECRET_SCAN`
- Do not set this permanently in your shell profile
- Do not commit files with real secrets even with the override active

The override bypasses the scan only — it does not disable the `.env` write block
or the DEC-001 payload check. Those run regardless.

---

## Iron Rules

1. **Tests first.** `qa-agent` has a hard veto. No story is done without passing tests.
2. **QA before code review.** `/review` runs QA first — broken code doesn't get reviewed.
3. **Human approval.** Always show diffs before applying changes.
4. **Backlog everything.** Review findings that don't block merge → BACKLOG.md. Never lost.
5. **Decisions logged.** Every architecture choice → DECISIONS.md with a DEC-XXX number.
6. **NEXT.md is sacred.** End every session with the single most specific next action written.
7. **Append-only memory.** LEARNINGS.md and completed stories are never deleted.

---

## Security Review Cadence

The team tracks when `/security-review` was last run and flags it during `/standup` if overdue.

**Default:** 30 days (see `.claude/commands/standup.md` for the configured value — that file is the single source of truth)

**How to change the threshold:**
Edit the single value on the line marked `<!-- CONFIGURABLE: ... -->` in
`.claude/commands/standup.md`. That is the only integer you need to change — the logic
condition and flag message text both reference it as "the configured threshold".

**What triggers the flag:**
The `security-analyst-agent` standup block will set a `Blocked:` entry when either:
- The `## Last Security Review` line in `memory/STATE.md` reads `[Never run]`
- The date recorded there is more than the configured threshold (default: 30 days) before today

When within threshold, no flag is raised and the standup stays clean.

**Where review results are stored:**
- `memory/STATE.md` — `## Last Security Review` line (overwritten each run)
- `memory/LEARNINGS.md` — `## Security Review Log` section (append-only history)

---

## Setup

```bash
cd your-project
curl -fsSL https://raw.githubusercontent.com/thecoderbuddy/agile-team-skill/main/install.sh | bash
claude
```

Then run `/init` — agents scan your project (or use your description) and populate the memory files with real stories, a sprint goal, and your first next action.

The team is ready when `/init` writes to `memory/BACKLOG.md`.

---
> Source: [thecoderbuddy/agile-team-skill](https://github.com/thecoderbuddy/agile-team-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
