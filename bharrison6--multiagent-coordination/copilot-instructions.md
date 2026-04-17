## multiagent-coordination

> This configuration applies to all repositories in `C:\github\`.

# Cross-Agent Coordination Protocol

This configuration applies to all repositories in `C:\github\`.

**Owner:** dover
**Repository Root:** Contains multiple project repositories. Each repo has its own STANDARDS.md with project-specific technical rules.

## How to Use This Configuration

This file contains **ONLY cross-agent coordination protocol**. For project-specific technical rules, see each repository's `STANDARDS.md` file.

**When working in a specific repo:**
1. Navigate to that repo's directory
2. Read that repo's `STANDARDS.md` for all technical rules

**Shared integration workflows** (reusable, project-agnostic patterns established across projects):
- See `C:\GitHub\workflows\` — before implementing any new integration from scratch, check here first
- Current guides: Google Apps Script + clasp (`workflows\google-apps-script-clasp.md`)

## Cross-Agent Activity Logging

Multiple AI agents (Claude, Gemini, Codex) work in this repository. To coordinate effectively, all agents must log their activity.

### Before Starting Work

1. **Read `<repo>\.agent-log\changelog.md` by direct path** - Review recent changes by all agents. Always use a direct file Read by absolute path — do NOT rely on Glob or directory listings to confirm these files exist. On Windows, Glob can silently miss files in `.agent-log\` directories. Only if Read returns "file not found" can you safely assume the file is absent.
2. **Read `<repo>\.agent-log\handoffs.md` by direct path** - Check for pending tasks or blockers. Same rule: direct Read only, never Glob.
3. **Read repo standards** - Navigate to the specific repo and read its `STANDARDS.md`

### Root-Level vs Repo-Level Logging

- **Repo-level** (`<repo>\.agent-log\`): Use for all work within a specific repository
- **Root-level** (`C:\github\.agent-log\`): Use for:
  - Protocol/config changes (CLAUDE.md, GEMINI.md, AGENTS.md edits)
  - Cross-repo tasks spanning multiple repositories
  - System-level reviews or coordination work
- When logging to root, use `Repo: ROOT` instead of a repo name
- Root log uses **absolute paths**; repo logs use **paths relative to repo root**

### During Work

- Follow the technical standards defined in the repo's `STANDARDS.md`
- Check for duplicate work before implementing new features
- Maintain consistency with existing code patterns

### After Completing Work

1. **Log to `<repo>\.agent-log\changelog.md`** with this format:
   ```
   ### [YYYY-MM-DD HH:MM] GEMINI [ACTION_TYPE]
   - Description of changes
   - Repo: <repo-name>
   - Files modified: <paths from repo root>
   - Notes: Important context for other agents
   - PENDING: (optional) What needs follow-up
   ```

2. **Action types:** `[IMPLEMENT]`, `[REFACTOR]`, `[FIX]`, `[TEST]`, `[CONFIG]`, `[DOCS]`, `[REVIEW]`

3. **If handing off incomplete work:** Update `<repo>\.agent-log\handoffs.md` with:
   - What was completed
   - What's still pending
   - Any blockers or issues
   - Which agent should continue

4. **Agent log files are append-only — never overwrite their contents.** Always Read the file first, then add your new entry while preserving all existing content. Only create a file fresh if Read confirmed the file does not exist.

### Update Check Triggers

Re-read configuration files when:
- Another agent logs "updated standards" or "updated config" in changelog
- You see unfamiliar code patterns in existing files
- More than 24 hours since your last session
- Build errors suggest API changes

## Multi-Agent Best Practices

### Coordination

- **No Duplication:** Always check changelog before implementing new features
- **Clear Handoffs:** Use handoffs.md when leaving incomplete work
- **Log Everything:** Better to over-communicate than under-communicate
- **Respect Standards:** Follow each repo's STANDARDS.md strictly
- **Preserve Logs:** Never overwrite `changelog.md` or `handoffs.md`. Always Read first and append your entry. Overwriting destroys other agents' history and handoff data.

### Agent Roles

- **Claude CLI:** Terminal-based, navigates filesystem, sees full repo context
- **Claude IDE:** IDE-based, opened at repo or project level, reads CLAUDE.md
- **Gemini CLI:** Terminal-based, navigates filesystem, sees full repo context
- **Gemini IDE:** IDE-based, opened at repo or project level, reads GEMINI.md
- **Codex CLI:** Terminal-based, navigates filesystem, sees full repo context
- **Codex IDE:** IDE-based, opened at repo or project level, reads AGENTS.md

### Configuration Hierarchy

```
C:\github\
|-- CLAUDE.md                       <- Claude coordination protocol
|-- GEMINI.md (this file)           <- Gemini coordination protocol
|-- AGENTS.md                       <- Codex coordination protocol
|-- .agent-log\
|   |-- changelog.md                <- Root/system-level activity
|   \-- handoffs.md                 <- Root-level handoffs
|-- .agent-proposals\               <- Protocol change proposals
|-- scratch\                        <- Untracked: temp files, screenshots, debug scripts
|-- workflows\                      <- Shared integration guides (reusable across all repos)
|   \-- google-apps-script-clasp.md <- GAS + clasp: backend, web app, deploy automation
|
\-- <repo-name>\
    |-- STANDARDS.md                <- Repo-specific technical rules (READ THIS!)
    |-- CLAUDE.md                   <- Claude behavior for this repo
    |-- GEMINI.md                   <- Gemini behavior for this repo
    |-- AGENTS.md                   <- Codex behavior for this repo
    |-- .agent-log\
    |   |-- changelog.md            <- All activity for this repo
    |   \-- handoffs.md             <- Pending handoffs for this repo
    |
    \-- <project>\
        |-- CLAUDE.md               <- Redirect: points to root + repo configs
        |-- GEMINI.md               <- Redirect: points to root + repo configs
        \-- AGENTS.md               <- Redirect: points to root + repo configs
```

## Repository-Specific Standards

Each repository has its own `STANDARDS.md` defining technical rules, architecture patterns, and project-specific details.

**IMPORTANT:** Never assume standards are the same across repos. Always read the specific repo's STANDARDS.md before working in that repo.

## Changelog Format Examples

### Good Changelog Entry
```
### [2026-01-23 14:30] GEMINI [IMPLEMENT]
- Implemented new feature X with Y approach
- Repo: my-project
- Files: src/path/to/modified/file.java
- Notes: Follows repo standards, uses pattern Z
- PENDING: Needs testing
```

### Good Handoff Entry
```
### [2026-01-23 15:00] GEMINI → ANY
- Task: Complete feature X testing
- Status: Implementation done, tests written, not yet verified
- Blocker: Need access to Y
- Next Steps: Run integration tests, verify behavior, tune parameters
- Files: my-project/src/path/to/file.java
```

## When Config Files Change

If you update any configuration file (STANDARDS.md, GEMINI.md, etc.):
1. Log the update in changelog with `[CONFIG]` action type
2. Note specifically what changed
3. Other agents will re-read configs when they see this log entry

## Protocol Change Process

When you identify issues with the coordination protocol:
1. **Never edit protocol files directly** without human approval
2. **Create a proposal** at `C:\github\.agent-proposals\<agent-name>-<topic>.md`
3. **Log the proposal** in the appropriate changelog with `[REVIEW]` action type
4. **Wait for human approval** before implementing changes

---

**Remember:** This file is for cross-agent coordination only. For technical standards, always read the repo-specific STANDARDS.md file!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bharrison6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
