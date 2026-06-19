## career-ops

> AI-powered job search and application pipeline. Scans portals (LinkedIn, Indeed, Glassdoor, Greenhouse, Ashby, Lever, Workable, Welcome to the Jungle, Handshake, Reed UK), scores postings against the candidate's profile with A–G blocks, drafts tailored CVs and cover letters, runs interview prep with STAR+R stories, and logs everything to a Notion Applications tracker (auto-created on first run) plus a Notion dashboard view. The agent NEVER auto-submits — it stops before Submit/Send/Apply and waits for human approval. TRIGGER when the user types /career-ops, pastes a job URL, asks to scan job portals, requests a tailored CV or cover letter for a specific role, asks for interview prep for a named company, asks about application status, pipeline, or tracker, or mentions job search, job hunt, applications, recruiter, ATS, or career change. On first run the skill enters onboarding and prompts for CV upload, LinkedIn URL, portfolio URL, target job markets, target roles, scan hour, Bright Data API key, and Notion integration token; then bootstraps the user-layer files into the user's workspace, auto-creates the Notion Applications database with the canonical schema, and adds kanban plus by-stage dashboard views. SKIP for general resume formatting questions unrelated to a live job search, or for non-career-related work.


# Career-Ops — AI Job Search Pipeline

## What this skill does

```
scan portals → score postings (A–G) → draft tailored CV + cover letter → human approves → submit (you click) → Notion tracker logs everything
```

The agent **never auto-submits**. It fills forms, drafts answers, generates PDFs, prepares cover letters — but always stops before Submit / Send / Apply and waits for the user to confirm.

## When invoked

Default response to:

- The user types `/career-ops` (with or without arguments).
- The user pastes a job URL or JD into the chat.
- The user asks to "scan jobs", "find roles", "tailor my CV", "draft a cover letter", "evaluate this job", "prep for an interview at X", "show my pipeline", "log this application".

---

## Where the skill lives vs. where user data lives

The skill is **self-contained**. After install (`git clone https://github.com/marz1307/career-ops ~/.claude/skills/career-ops && cd ~/.claude/skills/career-ops && npm install`), this layout is on disk:

```
~/.claude/skills/career-ops/          ← ENGINE_DIR (this skill folder)
├── SKILL.md                          ← you are reading this
├── modes/*.md                        ← mode definitions
├── templates/                        ← portal config, CV template, states
├── config/profile.example.yml
├── *.mjs                             ← scan.mjs, generate-pdf.mjs, …
├── package.json                      ← node deps (playwright, js-yaml, dotenv)
└── node_modules/                     ← created by `npm install`
```

ENGINE_DIR is **read-only from the user's perspective**. Updates land here; nothing user-specific is written here.

The **WORKSPACE** is the directory the user runs `/career-ops` from. That's where personal files go:

```
<wherever the user cd'd to>          ← WORKSPACE
├── cv.md
├── article-digest.md
├── config/profile.yml
├── modes/_profile.md
├── portals.yml
├── .env                              ← BRIGHTDATA_API_KEY, NOTION_TOKEN
├── data/                             ← applications.md, pipeline.md
├── reports/
├── output/                           ← generated PDFs
└── interview-prep/
```

**Convention used throughout this skill:**
- "Read `modes/X.md`" → read from **ENGINE_DIR/modes/X.md**.
- "Read / write `cv.md` / `config/profile.yml` / `portals.yml` / `data/*` / etc." → operate on the **WORKSPACE** copy.
- Bash invocations like `node scan.mjs` → `node $ENGINE_DIR/scan.mjs` with cwd = WORKSPACE so the script reads the user's portals.yml and writes to data/.

To get ENGINE_DIR, the agent uses the absolute path of this SKILL.md (Read tool resolves it). On most systems it's `${HOME}/.claude/skills/career-ops` (POSIX) or `${USERPROFILE}\.claude\skills\career-ops` (Windows).

---

## Step 1 — Onboarding check

The WORKSPACE = the user's current working directory.

Silently check whether the user-layer files exist in the WORKSPACE:

| Required file / state | If missing |
|---|---|
| `cv.md` | Enter onboarding (Step 2). |
| `config/profile.yml` | Enter onboarding (Step 2). |
| `modes/_profile.md` | Copy `$ENGINE_DIR/modes/_profile.template.md` → `WORKSPACE/modes/_profile.md` silently. |
| `portals.yml` | Enter onboarding (Step 2). |
| `.env` with `NOTION_TOKEN` | Enter onboarding (Step 2). |
| `config/profile.yml → notion.applications_db_id` | Enter onboarding (Step 2). |

If ALL exist, jump to Step 6 (route the user's request).

Before starting onboarding, confirm the workspace location:

> "I'll set up your career-ops workspace in `${cwd}`. That's where your CV, profile, tracker, and generated PDFs will live. Reply 'yes' to use this folder, or give me a different absolute path."

If the user gives a different path: `mkdir -p <path>`, then use it as WORKSPACE for the rest of the session.

## Step 2 — Onboarding: collect inputs

Use **AskUserQuestion** to collect the onboarding inputs. The tool caps at 4 questions per call, so split into two consecutive batches:

> "Welcome to career-ops. To set up your personal pipeline I need a few things. Nine questions split across three batches (4 + 4 + 1)."

**Batch 1 (questions 1–4):**

1. **CV** — `header: "Your CV"`. Multi-select: false. Options:
   - "Paste my CV text in the next message"
   - "Give me a file path (PDF / DOCX / MD) on this machine"
   - "Use my LinkedIn profile only"
   - "Draft from scratch — I'll answer questions"

2. **LinkedIn** — `header: "LinkedIn"`. Multi-select: false. Options:
   - "I'll paste my LinkedIn URL in the next message"
   - "I don't have a LinkedIn / skip"

3. **Portfolio** — `header: "Portfolio"`. Multi-select: false. Options:
   - "I'll paste my portfolio / personal site URL in the next message"
   - "I'll paste my GitHub URL in the next message"
   - "Skip — no portfolio"

4. **Target markets** — `header: "Markets"`. Multi-select: **true**. Options:
   - "United Kingdom"
   - "European Union (broad)"
   - "United States"
   - "Remote (any region)"

**Batch 2 (questions 5–8):**

After Batch 1, the workspace has identity + direction. Batch 2 fills in roles, scraping infrastructure choices, persistence, and timing.


5. **Target roles** — `header: "Target roles"`. Multi-select: false. Options:
   - "I'll list my target role titles in the next message (e.g. 'Senior Backend Engineer, Staff Platform Engineer')"
   - "Infer from my CV"

6. **Bright Data API key** — `header: "Bright Data"`. Multi-select: false. Options (free-first):
   - "Skip — use free ATS APIs only (Greenhouse / Ashby / Lever / Workable, ~150 companies). No LinkedIn / Glassdoor / Indeed / Monster coverage."
   - "I have a Bright Data API key — adds the auth-walled portals back"
   - "What's Bright Data? Explain first"

7. **Notion integration** — `header: "Notion"`. Multi-select: false. Options:
   - "I have a Notion integration token — I'll paste it next"
   - "Walk me through creating one"
   - "Skip Notion — use local tracker only"

8. **Scan schedule** — `header: "Run time"`. Multi-select: false. Options:
   - "07:00 local — catch overnight postings before recruiters open inboxes"
   - "12:30 local — lunchtime sweep"
   - "18:00 local — end-of-day catch-up"
   - "Custom — I'll specify the hour next"
   - "Don't schedule — I'll run scans manually"

**Batch 3 (question 9):** — runs only after Batch 2 because Firecrawl install can take a few minutes on first run, and the user gets cleaner feedback if it's its own batch.

9. **Firecrawl (clean Step −1 coherence + scraping)** — `header: "Firecrawl"`. Multi-select: false. Options (free-first):
   - "Self-host (Docker required, no keys) — recommended. Clean Step −1 coherence + full local extraction."
   - "Skip — use Playwright + WebFetch (always works, no Docker)"
   - "Cloud (paste API key) — easiest if Docker isn't available, paid per scrape"

If "Explain first" on Bright Data:

> "Bright Data is a web-data service. With a key, the scanner can hit LinkedIn, Indeed, Glassdoor, and Welcome to the Jungle through a real browser (more coverage, more reliable). Without a key, the scanner falls back to free ATS JSON endpoints (Greenhouse, Ashby, Lever, Workable). Sign up at https://brightdata.com if you want one. You can always add the key later."

If "Walk me through" on Notion:

> "1. Go to https://www.notion.com/profile/integrations and click 'New integration'. Name it 'career-ops', leave capabilities at the defaults. Click Save.
> 2. Copy the 'Internal Integration Token' (starts with `ntn_`).
> 3. Pick a Notion page where you want the tracker to live. Open the page, click ··· → Connections → Add connections → select 'career-ops'.
> 4. Paste the token here. Tell me the URL of the parent page so I can create the tracker inside it."

## Step 3 — Onboarding: gather the actual data

After the user answers, prompt for the actual values in plain chat (one message per item). For each value received:

### CV (writes to `WORKSPACE/cv.md`)
- Pasted text → write directly as clean markdown (Summary, Experience, Projects, Education, Skills).
- File path → Read the file, convert to clean markdown. For PDF, use the `pdf` skill or `pdftotext` via Bash.
- LinkedIn only → fetch the LinkedIn page (Bright Data MCP if a key was provided, otherwise WebFetch) and synthesise a CV.
- Draft from scratch → ask 5–8 targeted questions, then write `cv.md`.

### LinkedIn URL
- Write to `WORKSPACE/config/profile.yml → candidate.linkedin`.

### Portfolio URL
- Write to `WORKSPACE/config/profile.yml → candidate.portfolio_url` (or `candidate.github`).

### Target markets
- Write to `WORKSPACE/config/profile.yml → target_markets: [<list>]`.
- Seed `WORKSPACE/portals.yml.location_filter.allow` and `always_allow`.

### Target roles
- Pasted titles → write to `target_roles.primary: [<list>]` AND seed `portals.yml.title_filter.positive`.
- "Infer from CV" → extract role titles from the most recent 2 positions in `cv.md`. Confirm before saving.

### Bright Data key
- Write `BRIGHTDATA_API_KEY=<value>` to `WORKSPACE/.env`. Never to `config/profile.yml`.

### Firecrawl
Three branches based on the answer to question 9:

**Self-host (Docker required, no keys) — DEFAULT**:
1. Detect Docker availability — run `docker info` via Bash. If it fails (Docker not installed, daemon not running, plugin missing), surface the error and re-ask the user with two follow-up options: "Cloud key" or "Skip — Playwright fallback."
2. If Docker is available, run the install script:
   ```bash
   bash $ENGINE_DIR/scripts/install-firecrawl.sh --workspace "$WORKSPACE"
   ```
   On Windows / PowerShell:
   ```powershell
   & "$ENGINE_DIR\scripts\install-firecrawl.ps1" -Workspace "$WORKSPACE"
   ```
3. The script clones Firecrawl into `~/.career-ops/firecrawl`, runs `docker compose up -d`, waits for `/health` on `:3002`, and writes `FIRECRAWL_URL=http://localhost:3002` into `WORKSPACE/.env`.
4. On success, tell the user the Firecrawl admin UI is at `http://localhost:3002/admin/CHANGEME/queues` and that they can stop it any time with `bash $ENGINE_DIR/scripts/install-firecrawl.sh --stop`.

**Cloud (paste API key)**:
- Ask the user to paste their Firecrawl API key. Write `FIRECRAWL_API_KEY=<value>` to `WORKSPACE/.env`. Do NOT set `FIRECRAWL_URL`.

**Skip — Playwright fallback**:
- Don't touch `.env`. The fetch-chain (`providers/_fetch-chain.mjs`) will skip the Firecrawl tier and try Bright Data → Playwright → WebFetch.

After ANY of the three branches, the `.mcp.json` at the skill root auto-registers the `firecrawl` MCP server next time Claude Code reloads. The server simply no-ops if neither `FIRECRAWL_URL` nor `FIRECRAWL_API_KEY` is set — no harm done.

### Notion token + parent page
- Write `NOTION_TOKEN=<value>` to `WORKSPACE/.env`.
- Ask for the **parent page URL**. Extract the page ID (32-char hex).
- Auto-create the Applications database (Step 4).

### Scan schedule
- If preset or "Custom", confirm the timezone (default to `config/profile.yml → location.timezone`) and cadence (default: weekdays). Write to `config/profile.yml`:
  ```yaml
  schedule:
    scan_hour_local: "07:00"
    timezone: "Europe/London"
    days: [1, 2, 3, 4, 5]
  ```
- Call `/schedule` to register a recurring `/career-ops scan`. If unavailable, surface a cron / Task Scheduler snippet.
- "Don't schedule" → skip.

### profile.yml essentials
After the CV exists, copy `$ENGINE_DIR/config/profile.example.yml` → `WORKSPACE/config/profile.yml` and fill from what the user gave you. Prompt once for:
- Full name + email + phone + location + timezone.
- Salary target range and currency.

### portals.yml
Copy `$ENGINE_DIR/templates/portals.example.yml` → `WORKSPACE/portals.yml`. Update `title_filter.positive` from target roles and `location_filter` from target markets.

### modes/_profile.md
Copy `$ENGINE_DIR/modes/_profile.template.md` → `WORKSPACE/modes/_profile.md`.

### data/applications.md
Create with the standard header in `WORKSPACE/data/applications.md`:

```markdown
# Applications Tracker

| # | Date | Company | Role | Score | Status | PDF | Report | Notes |
|---|------|---------|------|-------|--------|-----|--------|-------|
```

### article-digest.md (optional)
> "Have you published any projects / articles / case studies I should know about? Paste links or descriptions and I'll add them to your proof points. Skip if not."

If they give content, write to `WORKSPACE/article-digest.md`.

## Step 4 — Auto-create the Notion Applications tracker

### Step 4a — Create the database

Call `notion-create-database` with:
- `parent.page_id` = the page ID the user gave you.
- `title` = "🎯 Applications".
- `properties` = the full schema (see `$ENGINE_DIR/modes/notion-tracker.md → Applications DB — schema`).

Load via ToolSearch if deferred (`select:mcp__*notion-create-database`).

On success, write the returned `database.id` and `data_source_id` to `WORKSPACE/config/profile.yml`:

```yaml
notion:
  applications_db_id: "<32-hex-no-dashes>"
  applications_data_source_id: "<UUID-with-dashes>"
```

### Step 4b — Create the dashboard views

Add three views via `notion-create-view`:
1. **Pipeline board** (kanban grouped by Stage) — primary view.
2. **By score** (table sorted by Match score DESC, filtered to Stages 2–3).
3. **Active interviews** (board grouped by Stage, filtered to Stages 5–9).

If `notion-create-view` isn't available, instruct the user to add them manually.

### Step 4c — Confirm

Print the database URL Notion returned.

## Step 5 — Confirm setup

> "Setup complete. Your workspace at `${WORKSPACE}`:
>
> - `cv.md` — your canonical CV
> - `config/profile.yml` — identity, targets, comp, Notion IDs
> - `modes/_profile.md` — archetypes and framing
> - `portals.yml` — scanner config
> - `article-digest.md` — proof points
> - `.env` — local secrets (gitignored locally if you `git init` here)
> - Notion Applications DB at <URL>, with Pipeline / By score / Active interviews views
>
> You can now:
> - Paste a job URL to evaluate it
> - Run `/career-ops scan` to search portals
> - Run `/career-ops pdf` to generate a tailored CV
> - Say 'change my target roles to X' to tweak anything
>
> Tip: from any new terminal, `cd ${WORKSPACE}` first, then type `/career-ops`."

## Step 6 — Route the user's request

Route based on what the user typed AFTER `/career-ops`:

| User input | Mode |
|---|---|
| URL or JD pasted in chat | auto-pipeline → `$ENGINE_DIR/modes/auto-pipeline.md` |
| `evaluate <url>` / `oferta <url>` | `modes/oferta.md` |
| `scan` | `modes/scan.md` |
| `pipeline` | `modes/pipeline.md` |
| `pdf` / `cv` | `modes/pdf.md` |
| `apply` / `fill form` | `modes/apply.md` |
| `interview-prep <company>` | `modes/interview-prep.md` |
| `contacto` / `linkedin outreach` | `modes/contacto.md` |
| `deep <company>` / `research <company>` | `modes/deep.md` |
| `tracker` / `status` | `modes/tracker.md` |
| `patterns` | `modes/patterns.md` |
| `followup` | `modes/followup.md` |
| `batch` | `modes/batch.md` |
| `ofertas` / `compare` | `modes/ofertas.md` |
| `training <course>` | `modes/training.md` |
| `project <project>` | `modes/project.md` |
| no args | print the help menu |

For each mode, read `$ENGINE_DIR/modes/<mode>.md` and follow its instructions. Always read `$ENGINE_DIR/modes/_shared.md` first (system defaults), then `WORKSPACE/modes/_profile.md` (user overrides) before executing.

When a mode tells you to run a script (e.g. `node generate-pdf.mjs`), run it as:

```bash
cd "$WORKSPACE" && node "$ENGINE_DIR/<script>.mjs" <args>
```

This way the script's cwd is the workspace (so it reads `portals.yml`, writes to `data/`, etc.) while the script code itself comes from the engine.

## Ethical rules — non-negotiable

1. **Never auto-submit.** Stop before Submit / Send / Apply.
2. **Honour the score floor.** Below `triage.score_floor` (default 70/100) → `Not pursuing`, not drafted.
3. **Quality over speed.** Discourage low-fit applications.
4. **Respect recruiters' time.**
5. **Verify postings are live with Playwright before drafting** (batch / headless is the only exception).

## What this skill does NOT do

- Does not submit applications on behalf of the user.
- Does not impersonate the user in interviews or live recruiter conversations.
- Does not store secrets anywhere except `WORKSPACE/.env`.
- Does not send messages, schedule meetings, or commit code without explicit confirmation.

---
> Source: [marz1307/career-ops](https://github.com/marz1307/career-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
