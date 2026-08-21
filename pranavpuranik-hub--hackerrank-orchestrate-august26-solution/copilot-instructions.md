## hackerrank-orchestrate-august26-solution

> HackerRank Orchestrate (August 2026) — Message Notification Router

# AGENTS.md

HackerRank Orchestrate (August 2026) — Message Notification Router

This file is the single source of truth for any AI coding agent working in this repo: Claude Code, OpenAI Codex CLI / Codex Cloud, Gemini CLI, Cursor, Windsurf, opencode, Aider, goose, Factory, RooCode, JetBrains Junie, GitHub Copilot, Devin, or any other AGENTS.md-aware tool.

Read this file in full before taking any action. Obey it exactly unless the user or platform provides higher-priority instructions.

---

## 0. TLDR For The Agent

On every session start, do this in order:

1. Read this file completely.
2. Check the log file path in §2.
3. If it contains a line starting with `AGREEMENT RECORDED:` that matches the current repo root, skip onboarding and go to §4.
4. Otherwise, run the onboarding flow in §3.
5. For every user turn after onboarding, append a summary entry to the log file using the format in §5.
6. When building, testing, or packaging the solution, follow the project contract in §6.

Do not skip logging, rewrite old log entries, or modify the onboarding gate. Sub-agents and worktrees use the same log file.

---

## 1. What This Repo Is

This is a starter repo for the **HackerRank Orchestrate** 24-hour hackathon challenge: **Message Notification Router**.

Participants must build an AI-powered system for WhatsApp. For every incoming multimodal message in `dataset/messages.csv`, the system decides whether the message should:

- `notify`: interrupt the user now
- `digest`: wait for later
- `mute`: be suppressed as low-value, repetitive, unwanted, suspicious, or unsafe

The system should use the provided user, group, business, historical message, image, voice-note, and interaction data to make personalized routing decisions across text, image posters/screenshots, and voice notes.

The final submission must produce `output.csv` with:

```text
message_id,action,message_type,reason,confidence,evidence_message_ids
```

Read `problem_statement.md` for the full participant-facing specification.

---

## 2. Log File — Location And Lifecycle

The log file lives outside this repository so it survives branch switches, worktrees, and cleanup.

| Platform | Path |
|---|---|
| macOS / Linux | `$HOME/hackerrank_orchestrate_august26/log.txt` |
| Windows | `%USERPROFILE%\hackerrank_orchestrate_august26\log.txt` |

Rules:

- Create the file if missing, including the parent directory.
- Never commit or add the log file to git.
- Append only. Do not rewrite, reorder, or delete prior entries.
- Share this same log across all agents, sub-agents, and worktrees.
- Never log secrets. Redact API keys, tokens, cookies, private keys, and sensitive PII.

---

## 3. Onboarding Flow

Run this flow only if the log file has no `AGREEMENT RECORDED:` line for the current repo root. On later sessions, skip to §4.

### 3.1 Greeting

Open with a short, warm message. Example:

```text
Welcome to HackerRank Orchestrate. You have 24 hours to design, build, and ship a Message Notification Router for WhatsApp. Before we start, I need to walk you through the ground rules and get you set up. This takes about a minute.
```

Compute and display:

- Current system time, local timezone, ISO 8601.
- Time remaining until the challenge ends. Use the configured challenge end date if one is provided by the platform or README. If no challenge end date is present, say that the end time is not configured.
- Results announcement time, if provided by the platform or README.

If the current time is past the challenge end, say so plainly and ask whether the user is practicing, reviewing, or re-running tests. Do not block further work.

### 3.2 Rules — Recite These Verbatim

1. This is a **solo** challenge. You must be the author of the submission.
2. You may use any IDE, AI assistant, or tool to help you build. The deliverable is what your system can do, not how you wrote it.
3. Your system must conform to the project contract in §6 so it can be evaluated.
4. Never commit secrets. Use environment variables and a `.env` file if needed.
5. Logging of every conversation turn to the file in §2 is mandatory and cannot be disabled.
6. Submissions are made on the HackerRank Community Platform or as otherwise instructed by HackerRank.

### 3.3 Collect The Agreement

Ask the user to reply with the exact string `I agree` case-insensitively. Do not proceed until they do.

### 3.4 Record The Agreement

Append this block to the log file, then continue:

```text
## [ISO-8601 TIMESTAMP] ONBOARDING COMPLETE

AGREEMENT RECORDED: <repo_root_absolute_path>
Agent: <agent_name_or_unknown>
Language: js | ts | py | custom:<name>
System Time: <ISO-8601 local time with tz>
Time Remaining: <Xd Yh Zm, or not configured>
```

The repo root must match exactly so agreements do not leak across unrelated clones.

---

## 4. Normal Session Start

If onboarding is already complete for this repo root:

1. Append a short `SESSION START` entry using §5.1.
2. Greet the user briefly and surface the remaining time, or say the challenge end time is not configured.
3. If fewer than 2 hours remain, remind them to submit soon.
4. Proceed with the user's request.

---

## 5. Log Format

### 5.1 Session Start Entry

```text
## [ISO-8601 TIMESTAMP] SESSION START

Agent: <agent_name_or_unknown>
Repo Root: <absolute_path>
Branch: <git_branch_or_unknown>
Worktree: <worktree_path_or_main>
Parent Agent: <parent_agent_name_or_none>
Language: <js|ts|py|custom:name>
Time Remaining: <Xd Yh Zm, or not configured>
```

### 5.2 Per-Turn Entry

Append after every user message you respond to:

```text
## [ISO-8601 TIMESTAMP] <short title, max 80 chars>

User Prompt (verbatim, secrets redacted):
<exact user message, with secrets replaced by [REDACTED]>

Agent Response Summary:
<2-5 sentences: what was done, why, and any important decision>

Actions:
* <file edited / command run / tool invoked>

Context:
tool=<agent_name>
branch=<git_branch_or_unknown>
repo_root=<absolute_path>
worktree=<worktree_path_or_main>
parent_agent=<parent_name_or_none>
```

### 5.3 Sub-Agent And Worktree Rules

- Sub-agents must log their own entries using the same file.
- Set `parent_agent=` to the parent agent's name.
- Worktrees use the same shared log file, not a per-worktree copy.

### 5.4 What Not To Log

- API keys, tokens, session cookies, OAuth codes, or private keys.
- Sensitive PII.
- Full contents of large files or binary blobs. Reference by path instead.

---

## 6. Project Contract

### 6.1 Dataset Contract

Participant-facing files are inside `dataset/`.

```text
dataset/
├── messages.csv
├── output.csv
├── sample_messages.csv
├── users.csv
├── groups.csv
├── group_members.csv
├── business_accounts.csv
├── user_business_history.csv
├── message_history.csv
├── message_events.csv
├── images.csv
├── voice_notes.csv
├── daily_notification_summary.csv
└── media/
    ├── images/
    └── audio/
```

Organizer-only files, if present, live outside `dataset/` and must not be used for predictions.

### 6.2 Required Output

The solution must write `output.csv` with the exact columns below:

```text
message_id,action,message_type,reason,confidence,evidence_message_ids
```

There must be exactly one prediction row for every `message_id` in `dataset/messages.csv`.
Use `none` in `evidence_message_ids` when no useful historical evidence exists.

### 6.3 Constraints That Make The Submission Evaluable

- Be runnable from the terminal.
- Read the provided files from `dataset/`.
- Do not use organizer-only files or hardcoded labels.
- Keep behavior deterministic where possible.
- Read secrets from environment variables only.
- Include clear setup and run instructions in the submitted code package.

### 6.4 Reasonable Entry Points

There is no required language. If you use Python, `code/main.py` is a good entry point. If you use another language, document the run command clearly in your submitted README.

---

## 7. Cross-Platform And Agent-Compatibility Notes

- Resolve the log path using the platform home directory. Do not hardcode a user path.
- Write logs in UTF-8 with `\n` line endings.
- Do not assume bash. Prefer language-native APIs when possible.
- Keep tool-specific config minimal and point back to this `AGENTS.md`.
- If a nested `AGENTS.md` exists, the closest one wins for files inside that sub-project, but §2 and §5 remain global.

---

## 8. Quick Checklist For The Agent

Before responding to any user message, confirm:

- [ ] I have read this file in this session.
- [ ] I know whether onboarding is required.
- [ ] I know how much time is left, or that the end time is not configured.
- [ ] I will append a §5.2 entry after this turn.
- [ ] I will not log secrets.
- [ ] I will preserve the output contract in §6.

---
> Source: [PranavPuranik-hub/hackerrank-orchestrate-august26-solution](https://github.com/PranavPuranik-hub/hackerrank-orchestrate-august26-solution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
