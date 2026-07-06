## obsidian-para-gtd-ai

> This file provides context about this Obsidian vault for Claude Code.

# CLAUDE.md

This file provides context about this Obsidian vault for Claude Code.

> **First time here?** Run `/setup` to personalize this vault — it takes about 2 minutes and fills in all the placeholders below.

---

## Session Startup

**MANDATORY: Read these files immediately at the start of every session, before doing anything else. Do not skip, even in short sessions.**

1. `04_RESOURCES/Context/company-context.md` — Company, customers, market position
2. `04_RESOURCES/Context/product-context.md` — Product capabilities, architecture, value proposition
3. `04_RESOURCES/Context/process-context.md` — Team process, tools, rhythms
4. `04_RESOURCES/MEETINGS.md` — All recurring meetings: participants, purpose, agenda, vault paths

This context is essential for all work in this vault. Do not wait to be asked. Do not skip based on session length or topic.

---

## Vault Context

**Owner**: [YOUR NAME] — [YOUR ROLE]
**Primary Focus**: Professional knowledge management, work tracking, and AI-assisted decision-making
**Key Work Activities**: [e.g., Management, Strategy, Product, Engineering, Sales — customize this]
**Product / System**: [YOUR PRODUCT OR SYSTEM — e.g., a software platform, internal tool, or domain you manage]
**Vault Scope**: Professional work only
**Language Note**: [YOUR PREFERRED LANGUAGE — e.g., US English / German / both]

---

## Company & Team Structure

**Company**: [YOUR COMPANY NAME]
**My Role**: [YOUR ROLE] reporting to [YOUR MANAGER NAME]

**Key Contacts:**
- **[YOUR MANAGER NAME]** — [their role]
- **[TEAM LEAD / ENGINEERING MANAGER NAME]**: Key collaborator for technical scoping and team alignment.
- *Add others as needed*

**Direct Reports** *(if applicable — used by meeting and review skills)*:
- [Name 1] — [Role]
- [Name 2] — [Role]
- *Add or remove as needed*

---

## Organization

The vault uses a 6-folder structure:
- **00_INBOX**: Entry point for all new notes and captures.
- **01_ACTIVE**: Operational layer — Daily Notes, FOCUS, and IDEAS live here.
  - **Daily Notes/**: Current daily notes before archiving.
- **02_PROJECTS**: Active projects with a deadline and specific goal.
- **03_AREAS**: Ongoing responsibilities without deadlines.
  - **Organization/**: Recurring meetings and organizational formats, each in its own thematic subfolder. **NEVER save meeting notes directly in `03_AREAS/Meetings/` — always use a thematic subfolder.**
  - **People/**: Notes organized by relationship:
    - `People/Team/[Name]/` — 1:1s and notes for direct reports
    - `People/Peers/` — Notes for peer colleagues
    - `People/Leadership/` — Leadership contacts
  - **Learning/**: Ongoing learning and reading.
- **04_RESOURCES**: Reference materials, templates, and context files.
  - **Templates/**: Note templates (Daily Note, Meeting Preparation, Manager 1on1, Weekly Note)
  - **Context/**: Company, product, and process context files
  - **Weekly Reviews/**: Weekly synthesis notes
- **05_ATTACHEMENTS**: Storage for images, PDFs, and other binary files.
- **06_ARCHIVE**: Completed projects and inactive items.
  - **Daily Log**: Archived daily notes organized by `YYYY/MM_Month`

---

## Obsidian Vault Conventions

- Always read existing templates and frontmatter patterns in the vault BEFORE creating or editing notes. Match the existing tag names, template structure, and folder hierarchy exactly.
- Structure: Projects (02_PROJECTS) = time-bound deliverables, Areas (03_AREAS) = ongoing responsibilities, Resources (04_RESOURCES) = reference material. When unsure, check existing folder contents.
- **Meeting File Placement**: NEVER save a meeting note directly in `03_AREAS/Meetings/`. Always place it in `03_AREAS/Organization/[Thematic Subfolder]/`. Check existing subfolders first; create a new one only if no suitable folder exists.

---

## Project Structure Conventions

Every project in `02_PROJECTS/` follows a standard structure. Full reference: [[project-structure-conventions]].

**Standard folders:** `context/` · `resources/` · `output/` · `archive/` · `iterations/`

**Standard files:**
- `CLAUDE.md` — project context for Claude (goal, role, key files, working principles, session summary)
- `PLAN.md` — execution plan with phases, open decisions, next steps (version-stamped)
- `README.md` — human-readable overview

**`iterations/`** — one file per session (`iteration-01.md`, `iteration-02.md`, ...) for session continuity. Claude reads the latest iteration to restore context at session start.

---

## User Preferences

- **Tone**: Casual and compact. No fluff — text must be readable and efficient.
- **Language/Dialect**: [YOUR LANGUAGES — e.g., English / German / both]
- **Daily Notes**: Stored in `01_ACTIVE/Daily Notes`, archived monthly to `06_ARCHIVE/Daily Log/YYYY/MM_Month`.

---

## Custom Instructions

- **Memory Maintenance**: When updating `CLAUDE.md`, NEVER append an "Updates" section with a date. ALWAYS rewrite the relevant sections to integrate the new information seamlessly. The file must remain a clean, unified source of truth.
- **Linking Strategy**: Use `[[WikiLinks]]` for all internal references to ensure graph connectivity. **CRITICAL**: NEVER wrap `[[WikiLinks]]` in backticks or code blocks.
- **Date & Time Formatting**: ALWAYS use ISO 8601 format (YYYY-MM-DD for dates, HH:mm:ss for times).
- **Auto-Update Trigger**: Whenever you refine your workflow — including changes to vault structure, folder organization, note templates, task management approach, meeting organization, or custom skills — Claude must automatically recognize this and propose updating `CLAUDE.md` to reflect the change. Only trigger for genuine structural or workflow changes, not routine note-taking or task capture. When triggered, propose the specific section(s) to update and ask for confirmation before writing.

---

## Workflow Rules

### File Handling — Never Delete, Always Archive

- **NEVER delete any vault file** — regardless of source or type. Always move using Bash `mv`.
- Files that have been processed or are no longer active are moved to `06_ARCHIVE/` — never deleted.
- **Archive folder structure:**
  ```
  06_ARCHIVE/YYYY/MM_Month/
    Daily Log/     — archived daily notes
    Notes/         — inbox captures, emails, raw notes
    Transcripts/   — meeting transcripts
    Weekly Reviews/ — weekly synthesis notes (if archived)
  ```
- Create subfolders with `mkdir -p` if they don't exist yet.

### Transcript Handling (process-meeting)

- **NEVER delete transcript files** — regardless of where they come from.
- **Always archive transcripts** to `06_ARCHIVE/YYYY/MM_Month/Transcripts/` using the standardized filename format: `YYYY-MM-DD_[Meeting-Title]_[Participants].[ext]`
- **Always insert a `Transcript:` link** in the meeting note (below Attendees), pointing to the archived file: `[[06_ARCHIVE/YYYY/MM_Month/Transcripts/YYYY-MM-DD_...]]`
- This applies to every transcript format (`.txt`, `.md`, `.vtt`, `.docx`) from any source location.

### Date Awareness

- Always confirm which date the user is referring to before processing daily reviews or morning plans. The user sometimes catches up on previous days. Do NOT assume 'daily review' means today.
- When referencing meetings or events, verify temporal claims (past/future/cancelled) against the user's notes rather than assuming.

### Task & Todo Management Rules

- Always write todos to persistent vault files (daily notes or dedicated todo files), NEVER use TodoWrite for anything that needs to persist.
- When asked to update a list, only ADD or MODIFY the specific items requested — NEVER delete existing items unless explicitly asked.
- After completing a task management request, STOP. Do not offer additional help or suggestions unless asked.
- **Section Anchoring Rule:** When inserting a task into a specific section of a daily note, always use the **target section's own header + the last existing item within that section** as the `old_string` anchor. NEVER anchor at the next section's header — this causes items to land in structural gaps between sections.

**Task Format Standards:**

All tasks follow a pipe-separated format with emoji category prefix:

| Category | Format |
|---|---|
| Own task (urgent/today) | `- [ ] 🔴 \| Topic \| Description \| [[Link]] \| Date` |
| Own task (this week) | `- [ ] 📅 \| Topic \| Description \| [[Link]] \| Date` |
| Waiting For | `- [ ] ⌛ Name \| Topic \| Description \| [[Link]] \| Due date` |
| Delegated (new today) | `- [ ] 👥 Name \| Topic \| Description \| [[Link]] \| Due date` |
| Delegated (migrated to Hub) | `- [>] 👥 Name \| Topic \| ...` |
| Agenda Item | `- [ ] 📌 @Name \| Topic \| Source \| [[Link]]` |
| Someday | `- Topic: short note` *(no checkbox — max 2-week horizon, then → IDEAS.md)* |

**Task sections in Daily and Weekly Notes:**
- **🔴 Heute** — due today or genuinely urgent
- **📅 Diese Woche** — planned for this week, not urgent today
- **⏳ Waiting For** — blocked on someone else; always include name + deadline
- **👥 Delegiert** — today's new delegations only (capture point). Once migrated to the person's Hub file, mark `[>]`.
- **📌 Agenda Items** — topics to raise in a specific future meeting; marked `[>]` once added to the person/meeting file
- **💭 Someday** — no checkbox, no pressure; migrated to [[IDEAS]] if not actioned within ~2 weeks

**Key files:**
- `[[FOCUS]]` (`01_ACTIVE/FOCUS.md`) — active strategic themes by category; not a task list
- `[[IDEAS]]` (`01_ACTIVE/IDEAS.md`) — long-horizon ideas and backlog; no weekly pressure

### Scope Discipline

- When given a specific task, complete exactly that task. Do not proactively reorganize, add sections, or expand scope unless asked.
- When asked to correct specific items in a document, make ONLY those corrections — do not restructure or add new content.

---

## Note Templates & Standards

### Template Files

Note templates are stored in `04_RESOURCES/Templates/` and should be used as the authoritative source for note structure:

- **Daily Note Template:** `04_RESOURCES/Templates/Daily Note.md`
- **Meeting Preparation Template:** `04_RESOURCES/Templates/Meeting Preparation.md`
- **Manager 1:1 Template:** `04_RESOURCES/Templates/Manager 1on1.md`
- **Weekly Note Template:** `04_RESOURCES/Templates/Weekly Note.md`
- **Project Structure Reference:** `04_RESOURCES/Templates/project-structure-conventions.md`

**Command Behavior:** When commands like `/morning-plan` create new Daily Notes or supporting documents, they MUST read the appropriate template file from `04_RESOURCES/Templates/` and use it as the base structure, replacing placeholders like `{{date:YYYY-MM-DD}}` with actual values.

### Frontmatter Standards

Apply appropriate YAML frontmatter to all notes based on type:

**Daily Notes:**
```yaml
tags: [daily]
```

**Meeting Preparation/Notes:**
```yaml
tags: [meeting, preparation, topic-name]
up: "[[Parent Note or Area]]"
```

**Project Notes:**
```yaml
tags: [project, topic-category]
up: "[[02_PROJECTS/Project Name]]"
related:
  - "[[Related Notes]]"
```

**Resource Notes:**
```yaml
tags: [resource, topic]
up: "[[04_RESOURCES]]"
source: URL or reference if applicable
```

**General Guidelines:**
- Use lowercase tags with hyphens for multi-word tags (e.g., `product-management`, not `Product Management`)
- Always include an `up` link to establish hierarchy
- Use `related` to connect associated notes
- Include `source` for external references (URLs, documents, etc.)
- **CRITICAL: Always wrap WikiLinks (`[[...]]`) in quotes in YAML frontmatter** to prevent YAML parsing errors

---

## Claude Code Tools Reference

When executing commands, use these tools:
- **Search files by name**: Glob tool (e.g., `**/*.md`)
- **Search file contents**: Grep tool
- **Read files**: Read tool
- **Edit files**: Edit tool
- **Create files**: Write tool
- **Move/delete files**: Bash tool (`mv`, `rm`)
- **Create folders**: Bash tool (`mkdir -p`)
- **Web search**: WebSearch tool
- **Fetch URLs**: WebFetch tool

---

## Custom Skills

The vault includes custom workflow skills invoked with slash commands.

**Available Skills:**

- **`/setup`** — One-time vault personalization. Run this first.
- **`/morning-plan`** — Daily agenda builder and task organizer
- **`/daily-review`** — End-of-day reflection and task migration
- **`/inbox-processor`** — Systematic inbox organization
- **`/weekly-synthesis`** — Strategic pattern finder for weekly reviews
- **`/thinking-partner`** — Critical friend for clarifying ideas
- **`/research-assistant`** — Deep research combining internal and external sources
- **`/prep-meeting`** — Meeting preparation: loads context, compiles topics, creates prep note
- **`/process-meeting`** — Turns a meeting or transcript into a structured vault note
- **`/de-aiify`** — US English filter and anti-fluff text editor
- **`/create-command`** — Workflow architect for creating new skills

**Skill Location:** `.claude/skills/[skill-name]/SKILL.md`

**Session Startup — load all skills proactively:**

At the start of every session, use the `Read` tool to load each skill listed above. Skills are used frequently — do not wait to be asked. Read each `SKILL.md` file so it is available for the session.

Skills to load every session:
- `.claude/skills/process-meeting/SKILL.md` — processes meeting transcripts into structured vault notes
- `.claude/skills/prep-meeting/SKILL.md` — prepares any meeting

---
> Source: [timominkus/obsidian-para-gtd-ai](https://github.com/timominkus/obsidian-para-gtd-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
