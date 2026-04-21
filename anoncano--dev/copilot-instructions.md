## dev

> Docs & project conventions — structure, locations, format, version control


# Docs & Project Conventions

> Workspace-level rules for documentation, rambles, and file organization.
> Applies when creating/editing .md files or project docs.

---

## Dev Structure

**Multi-project workspace.** Dev folder holds many projects; structure scales for each.

```
c:\dev\
├── projects\           ← Project code (one folder per project)
├── ai\                 ← AI-related
│   ├── cursor\         ← Cursor docs, rules
│   ├── chats\          ← AI chat exports
│   └── projects\       ← AI-specific notes per project
├── docs\               ← Human docs, rambles
└── .zips\              ← Archives, versions, pre-change, version-archives (per project)
```

**AI vs human:** AI does most of the code; human directs, reviews, and contributes. See [DEV_OVERVIEW.md](../docs/DEV_OVERVIEW.md).

---

## Rules Maintenance

**When the user instructs to add a rule, make something a rule, or update the rules** (e.g. "add a rule for that", "make that a rule", "update the rules so that..."):

1. **Update** `ai/cursor/rules/docs-conventions.mdc` with the new or changed rule.
2. **Do it** — do not just acknowledge; actually edit the file.
3. **Log** the change in ACTIVITY_LOG.md (timestamp, what, why, how).

The .cursor/rules version is a hard link to this file, so it updates automatically.

---

## Doc Locations

| Location | Purpose |
|----------|---------|
| `c:\dev\ai\cursor\docs\` | AI-created/edited docs — overviews, plans, TODOs, summaries |
| `c:\dev\docs\` | Human-authored docs, rambles |
| `c:\dev\ai\chats\` | AI chat exports |
| `c:\dev\ai\projects\` | AI-specific notes per project |

**Single docs location for AI:** All .md files created or edited by AI go in `c:\dev\ai\cursor\docs\`. Do not create a second docs folder.

---

## Cursor-Generated .md Timestamp

- **Every .md file** created or edited by Cursor must include a timestamp in the header:
  - Format: `**Generated:** YYYY-MM-DD HH:mm` (for new files)
  - Format: `**Last updated:** YYYY-MM-DD HH:mm` (for edits)
- Place at top of file, after title if present

---

## Doc Format (Summaries)

Every .md file should be as accurate, detailed, and comprehensive as possible. Include four summary levels:

| Summary | Purpose |
|---------|---------|
| **Short** | One line or brief — quick scan |
| **Long** | Paragraph or two — full context |
| **Dumb** | Plain language, no jargon — anyone can understand |
| **Technical** | Paths, conventions, details — for implementers |

### Summary Placement

- **Top of file:** Add links to summaries after the title/metadata:
  ```
  [Short](#short) · [Long](#long) · [Dumb](#dumb) · [Technical](#technical)
  ```
- **Bottom of file:** Place the four summary sections (`## Short`, `## Long`, `## Dumb`, `## Technical`) at the end of the document.
- Main content goes in the middle.

### Edit Fields (user input areas)

Markdown has no native form fields. Use **placeholder format** for areas where user input goes:

```
> [Edit: hint or example — replace this with your content]
```

- **Format:** `[Edit: hint]` — user replaces the bracketed text with their input
- **Use in:** PROJECT_SPEC.md, template question files, any .md where user fills in details
- **Example:** `> [Edit: your-project-name — lowercase, hyphenated]`

---

## Project Doc Structure

**Adding a new project:** Use `/dev-new-project` (prompts for name, details, template) or manually create folders in `projects/<project-name>/`, `ai/cursor/docs/<project-name>/`, `docs/rambles/<project-name>/`, `.zips/<project-name>/`. Add to README.md and OVERVIEW.md Projects table.

- **Project code** lives under `c:\dev\projects\<project-name>/` (one folder per project)
- **Project-specific docs** live under `c:\dev\ai\cursor\docs\<project-name>/` (lowercase, hyphenated folder name)
- **Features/functionality subfolder:** Each project has `ai/cursor/docs/<project>/features/` for what/why/how docs
- **3+ .mds rule:** When a project docs folder has 3 or more .md files, create a subfolder to organize them
- **Each project** is allowed **one README.md** in its root that links to needed files
- README.md should link to `c:\dev\ai\cursor\docs\<project>/OVERVIEW.md`

---

## Rambles

- **Location:** `c:\dev\docs\rambles\` — rambles live in docs
- **Structure:** `docs/rambles/<project-name>/` (one folder per project)
- Raw notes, ideas, and brain dumps go in the project's rambles folder

---

## .zips

- **All .zip files** live in `c:\dev\.zips\` unless needed inside a project (e.g. build artifact, deployment package)
- **Structure per project:**
  ```
  .zips/<project-name>/
  ├── archives/        — .zip files (not yet extracted)
  ├── extracted/       — history of .zip files that have been extracted
  ├── versions/        — prod version zips (v1.0.zip, v1.1.zip) — only when project is in prod
  ├── pre-change/      — archives + logs before changes to deployed prod
  └── version-archives/ — superseded version zips (see conditions below)
  ```
- **misc/** — Workspace-level zips (not tied to a project): `misc/archives/` and `misc/extracted/`. dev-zip and dev-zip-full outputs go here.
- When extracting: move the .zip from `archives/` to `extracted/` (or add a note) to track history

### Commands (`.cursor/commands/`)

| Command | Purpose |
|---------|---------|
| **dev-help** / **dev-cmds** | List all commands |
| **dev-zip** | Workspace snapshot (exclude .zips) → misc/archives |
| **dev-zip-full** | Full snapshot (include project .zips, exclude misc) → misc/archives |
| **dev-new-project** | Create project — starts form first; form has template + questions; Create button scaffolds project |
| **dev-project-form** | Start form server (localhost:3333) |
| **dev-kill** | Kill form server, dev servers (ports 3333, 3000, 5173, 8080); add --all for all node |

**Templates for dev-new-project:** [ai/cursor/docs/TEMPLATES.md](../docs/TEMPLATES.md) is the **single source of truth**. Template list, APIs, framework docs links are there. When a template is chosen, create a README with walkthrough, demo INSTRUCTIONS, and links to documentation. Use k3s, not Docker.

**Source of truth:** Commands read `c:\dev\README.md` and `.zips/README.md` for folder structure. Update README when structure changes. Log in ACTIVITY_LOG.md.

### Small rolling set (versions/)

- **Keep** the latest **3** version zips in `versions/` per project
- When a 4th version is deployed, move the oldest to `version-archives/`
- Example: v1.2, v1.3, v1.4 in `versions/` → deploy v1.5 → move v1.2 to `version-archives/`, keep v1.3, v1.4, v1.5

### version-archives/ — when to create

- **Create** when a project is in prod and the rolling set is full (3 versions in `versions/`); move the oldest to `version-archives/`
- **Condition:** Use the small rolling set above; archive superseded zips to keep `versions/` manageable
- **Optional:** Not required until a project has 4+ deployed versions

---

## Version Control (Projects Near Production)

**When the user says a project is near production or deployed:** start at v1 and follow these rules.

### Version Numbers
- Start at **v1.0** when first deployed to prod
- Increment: v1.1, v1.2, … (patch), v2.0 (major), etc.

### Before Changes to Deployed Prod
- **Always** create a pre-change archive and log **before** making any changes to the deployed prod version
- **Archive:** Zip current prod state → `.zips/<project>/pre-change/YYYY-MM-DD_HHmm_<brief-description>.zip`
- **Log:** Create/append to `ai/cursor/docs/<project>/version-log/` with what, why, how:
  - **What** — what changed
  - **Why** — rationale, user need
  - **How** — implementation, flow

### When a Version Goes to Prod
- Create a version zip: `.zips/<project>/versions/vX.Y.zip` (e.g. v1.0.zip)
- Add a version log entry in `ai/cursor/docs/<project>/version-log/`

### Version Log Location
- `c:\dev\ai\cursor\docs\<project>/version-log/` — changelog, what/why/how for each version and pre-change

---

## Doc-Only Mode

When the user asks for documentation work (overview, plan, TODO, rambles, etc.):
- **Do not code** — work only with .md and text files
- Create, edit, and organize documentation only

---

## No Code Until New Command

**When the user says "no code until new cmd given" or similar:**

- **Do not write, edit, or run code** until the user gives a new command that explicitly requests it.
- **Do not** modify .ts, .tsx, .js, .jsx, .py, .json (code files), or run terminal commands that change code/builds.
- **Allowed:** .md files, docs, rules, logs, answering questions, planning.
- **Log** the activation in `ai/cursor/docs/NO_CODE_LOG.md` — timestamp, that no-code mode is on, and that code resumes only when user gives a new command.
- **Resume code** only when the user explicitly asks for code (e.g. "implement X", "fix Y", "run Z", "add feature").

---

## Rule Check

**When the user asks for a rule check, or when performing a compliance audit:**

1. **Audit** the workspace against docs-conventions.mdc (structure, doc format, timestamps, .zips, project docs, rambles).
2. **Update** `ai/cursor/docs/COMPLIANCE_REPORT.md` with results — status table, fixes applied (if any), scope, and summaries.
3. **Timestamp** the report with **Last updated:** YYYY-MM-DD HH:mm.
4. Never report compliance without updating the report.

---

## Change Tracking

**Every change in `c:\dev` must be logged.** Nothing happens without a trace of what, why, and how.

### Log Format

| Field | Purpose |
|-------|---------|
| **What** | What changed (files, structure, content) |
| **Why** | Rationale, user need, or trigger |
| **How** | Implementation, flow, or method |

### Where to Log

| Context | Location |
|---------|----------|
| **Prod projects** (deployed/near prod) | `ai/cursor/docs/<project>/version-log/` |
| **Workspace-level** (docs, rules, structure, .zips) | `ai/cursor/docs/ACTIVITY_LOG.md` |
| **Project work** (non-prod) | `ai/cursor/docs/<project>/version-log/` or ACTIVITY_LOG.md as appropriate |

### When to Log

- Creating, editing, or deleting files in ai/cursor/, docs/, .zips/, or project roots
- Rule checks → update COMPLIANCE_REPORT.md (see Rule Check above)
- Security checks → version-log or ACTIVITY_LOG per scope
- Structural changes (new folders, moves, renames)

**ACTIVITY_LOG.md** lives at `ai/cursor/docs/ACTIVITY_LOG.md`. Append entries with **timestamp** (YYYY-MM-DD HH:mm), what, why, how. Follow doc format (header timestamp, summaries).

---

## Security Check

**When the user asks for a security check, or when doing project audits:**

1. **Run security checks** on all projects in `c:\dev\projects\` (every project folder)
2. **Store reports** in `ai/cursor/docs/<project>/SECURITY_REPORT.md` (one per project)
3. **Update** project OVERVIEW.md and Key Docs to link to SECURITY_REPORT.md
4. **Format:** Follow doc format rules — timestamp, [Short](#short) · [Long](#long) · [Dumb](#dumb) · [Technical](#technical), summaries at bottom
5. **Scope:** Dependencies (npm audit, pip check), secrets/hardcoded creds, env handling, CORS, auth patterns, .gitignore for .env
6. **Do not fix code** unless the user asks — report findings only

---

## Hub Ecosystem Conventions

- **Container system:** k3s. Do not over-emphasize (e.g. avoid "ONLY k3s" or excessive repetition)
- **Scope:** Help with development and structure. User handles pricing, post-production, business decisions
- **Hub-to-hub app share:** User-triggered only, not automatic. Share limits set by marketplace admin

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anoncano) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
