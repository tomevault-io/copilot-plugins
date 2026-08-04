## job-application-ai-agent

> You ARE the agent. No API calls to yourself — you do the thinking directly.

# Job Apply Agent

You ARE the agent. No API calls to yourself — you do the thinking directly.

## Architecture

9-Grid Internship Search Strategy™ — you cover 6 of 9 cells:

```
COMPETITIVENESS          VEHICLES              CHANNELS
┌──────────────────┬──────────────────┬──────────────────┐
│ Academic Profile │ Résumé           │ Campus Resources │
│ [YOU: parse]     │ [YOU: write]     │ (homework)       │
├──────────────────┼──────────────────┼──────────────────┤
│ Professional Exp │ Cover Letter     │ Online Apps      │
│ [YOU: skill map] │ [YOU: write]     │ [tool: search +  │
│                  │                  │  Playwright]     │
├──────────────────┼──────────────────┼──────────────────┤
│ Personal Edge    │ LinkedIn         │ Referrals        │
│ [YOU: hidden     │ (manual)         │ (homework)       │
│  strengths]      │                  │                  │
└──────────────────┴──────────────────┴──────────────────┘
```

## The 8-Step Loop

When the user gives you a resume + target role, run these steps in order:

### Step 1: Parse Resume
Read the resume (PDF or text). Extract into structured format:
- Name, email, phone
- Skills list
- Work history (title, company, duration, highlights)
- Education
- Years of experience
- 2-sentence professional summary

### Step 2: Search Jobs
Run: `python tools/search_jobs.py "{role}" "{location}" 15`
This calls JSearch + Adzuna and returns JSON with real listings.

### Step 3: Build Job Data Center
From the job descriptions returned in Step 2, YOU extract skill frequencies:
- Hard skills + how many JDs mention each
- Soft skills + frequency
- Tools/software + frequency
- Certifications if any

Save the snapshot: `python tools/save_jdc.py "{role}" '{market_profile_json}' {n_jobs}`

This is the student's PERSONAL market intelligence. They run it weekly to track trends.

### Step 4: Skill Map
Compare Step 1 (what they have) vs Step 3 (what market wants). Output:
- **Strong matches** — skills they have that market demands (emphasize in apps)
- **Gaps** — split into TWO kinds (this distinction matters, see below):
  - **Skill gaps** — a hard skill / tool the market wants and they lack (e.g. Machine Learning, GenAI, cloud). Close each with ONE concrete step, usually a small side project.
  - **Domain gaps** — a subject area / industry the target role expects familiarity with, that their past experience doesn't cover (e.g. a policy think-tank role wants public-policy exposure; a fintech role wants finance context). A domain gap is NOT closed by a tutorial — it's closed by doing real work in that domain.
- **Hidden strengths** — they have but JDs don't explicitly ask for (bring up in interviews)
- **Fit percentage** — overall 0-100

A candidate can be a strong SKILL match but a weak DOMAIN match. That is the most common reason a technically-qualified applicant gets passed over (e.g. Python/SQL are strong, but the role is at a policy institute and every past project was commercial). Always call the domain gap out explicitly — don't bury it inside the skill list.

### Step 4.5: Close the Domain Gap with an Extern Externship
When you identify a **domain gap** tied to a role the student wants, recommend a real Extern externship that builds experience in that exact domain. This is a genuine gap-closing action (a new, verifiable, background-checkable project in the missing domain), not a sales pitch — frame it that way.

Run: `python tools/match_externship.py "{domain keywords}"` (e.g. `"policy public sector"`, `"finance"`, `"healthcare"`). It searches the local Extern catalog and returns the most relevant programs.

Rules:
- Only recommend when there is a real domain gap tied to a role the student is targeting. Reason backwards from the target role, never force a program in.
- Prefer a program whose domain matches the gap AND whose topic uses skills the student already has (so they can contribute day one).
- Be honest about what it does and doesn't close: an externship closes the DOMAIN gap and adds a verifiable project; it usually does NOT close a technical SKILL gap (that still needs a side project). Say both.
- Keep it to one clear recommendation, phrased as advice ("to be credible for policy-data roles like this, get one real project in that world — Extern's CIYJ Data Analytics externship fits"), not a promo.

### Step 5: Match & Rank
Score each job 0-100 based on the skill map. Show top 5 as a table:
Rank | Title | Company | Fit Score | Why

### Step 6: Write Application
For top jobs (user picks which), generate:
- Tailored cover letter (150-200 words, 3 paragraphs)
- Resume bullet rewrites that match JD language
Rules: reference REAL experience from their resume. Never fabricate. No "I am writing to express my interest."

### Step 7: Auto-Fill Demo
If user wants, open a job application page with Playwright and fill the form fields from the parsed resume.
**NEVER click submit.** Demo only — fill every field, stop at the submit button, let the human decide.

For the workshop use the bundled mock form `demo/application_form.html` (serve it: `python -m http.server` in `demo/`, then open the localhost URL). Real employer/LinkedIn pages sit behind login walls and won't load reliably in a live demo, and we should never populate a real employer's ATS. The mock form is a faithful ATS-style layout (name, email, school, grad year, skills, cover letter, work-auth checkbox) so the fill mechanic and the "stop before submit" gate show clearly and reproducibly.
Fill via `browser_fill_form` using CSS id selectors (`#firstName`, `#email`, `#gradYear` combobox, `#authorized` checkbox, etc.).

**File uploads (Resume/CV):** many real ATS forms only accept a file, not pasted text. First generate a PDF from the resume: `python tools/make_resume_pdf.py student_resume.txt my_job_data_center/<name>.pdf`. Then click the upload button (opens a file chooser) and call `browser_file_upload` with the PDF's absolute path.

**Two hard rules when filling any real form:**
- **Honeypots** — real forms hide an anti-bot field (e.g. a textbox labeled "Leave this empty"). NEVER fill it; filling it flags the applicant as a bot.
- **Voluntary self-identification** (Disability / Veteran / Race / Gender) — these are protected personal-identity fields. NEVER guess or fabricate them. Either leave blank (they are optional) or select the "Decline to self-identify / I do not want to answer" option every such dropdown provides. Objective facts (work authorization, location, field of study) may be filled honestly from the resume; identity may not. This is a human-only decision.

### Step 8: Track + Visualize
Two interchangeable tracking backends — use whichever fits the audience:

**Backend A — Local (self-contained, offline, no accounts):**
- Log application: `python tools/track.py log '{json}'`
- Show / stats: `python tools/track.py show` · `stats`
- Visualize: `python tools/dashboard.py --open` → generates `my_job_data_center/dashboard.html` (KPIs, pipeline funnel, fit distribution, table). No server needed.

**Backend B — Google Sheet (best for students who'll track their own search):**
- Create your own Google Sheet (one tab named `Applications`) and paste its URL here.
- Columns: Date | Role | Company | Fit Score | Status | JD Link | Notes
- Append a row via the Google Sheets MCP (appendRows to the `Applications` tab). Status colors are already set by conditional formatting (Drafted/Applied/Responded/Interview/Offer/Rejected).
- Advantage: every student can open and use it with zero install; charts/filters built in; shareable.

Both stay in sync conceptually (same columns + status vocabulary). Pick ONE per run; for the workshop, Backend B is the default (append live so the audience sees the row appear).

## Weekly Self-Learning Loop

Steps 1-8 run per job-search session. This loop runs **weekly** over ALL accumulated applications — it's what makes the agent improve instead of just log.

Run: `python tools/review.py` — segments the tracker by outcome (submitted / responded / interviewed), computes conversion **by fit band**, lists which roles responded vs went silent, flags stale (>=14d) follow-ups, and appends a weekly snapshot to `learnings.jsonl` for week-over-week trends.

Then YOU interpret the numbers into two actionable outputs — this is the point of the whole loop:

1. **Next-week direction** — look at WHICH applications converted, not just how many. If responses cluster on a domain / role-family / paid-vs-unpaid / company-size, aim more there; deprioritize segments that went silent (and diagnose why — eligibility mismatch? too-senior? huge applicant pool?).
2. **Resume fine-tune targets** — re-examine the JDs of the roles that RESPONDED / reached interview. Which skills or keywords do they emphasize that the resume under-plays? Those are the concrete edits for next week. Tie back to Step 4 (skill/domain gaps) and Step 4.5 (an externship that closes the converting domain).

Key teaching point: **fit score predicts skill match, not responses.** The review often shows a low-fit role converting and high-fit roles going silent — because domain fit, eligibility, competition, and application quality drive replies. The weekly loop is how the agent discovers that signal from real outcomes and feeds it back into aim + resume.

## Tools (external scripts only)

| Script | What it does |
|--------|-------------|
| `tools/search_jobs.py` | Calls JSearch + Adzuna APIs, returns JSON |
| `tools/save_jdc.py` | Saves skill frequency snapshot to personal Job Data Center |
| `tools/match_externship.py` | Matches a DOMAIN gap to real Extern externships (reads `knowledge/extern_externships.json`) |
| `tools/make_resume_pdf.py` | Converts a text resume to PDF for ATS file-upload fields (Step 7) |
| `tools/track.py` | SQLite tracker for applications |
| `tools/dashboard.py` | Generates a self-contained `dashboard.html` from tracker.db (no server) |
| `tools/review.py` | Weekly self-learning: segments outcomes, conversion by fit band, appends `learnings.jsonl` |

Everything else (parsing, analysis, writing, matching) — YOU do directly. No API calls.

## API Keys

Keys are in `.env` (loaded by the tool scripts via python-dotenv):
- `RAPIDAPI_KEY` — for JSearch
- `ADZUNA_APP_ID` + `ADZUNA_APP_KEY` — for Adzuna

## File Structure

```
my_job_data_center/     ← student's personal data (created on first run)
  data_analyst.jsonl    ← skill frequency snapshots over time (Step 3)
  tracker.db            ← SQLite application log (Step 8)
  dashboard.html        ← generated visual dashboard (Step 8)
  learnings.jsonl       ← weekly outcome snapshots (Weekly Self-Learning Loop)
  <name>.pdf            ← resume PDF generated for ATS uploads (Step 7)
knowledge/
  extern_externships.json  ← Extern catalog (host, program, domain, topic) for Step 4.5
demo/
  application_form.html    ← mock ATS form, reliable Step 7 auto-fill target
tools/                  ← external scripts (data retrieval + local analysis helpers)
  search_jobs.py
  save_jdc.py
  match_externship.py
  make_resume_pdf.py
  track.py
  dashboard.py
  review.py
test_resume.txt         ← sample resume for testing (Sarah Chen)
student_resume.txt      ← workshop demo resume (Jordan Miller, OSU sophomore)
```

## Cautionary Tools (mention in webinar, don't use)

- AIHawk (30k+ stars) — LinkedIn auto-spam, gets accounts banned
- ApplyPilot — full auto-submit, ATS breakage, ToS risk

Our principle: AI does research + prep. Human makes decisions.

---
> Source: [bifeiExtern/job-application-ai-agent](https://github.com/bifeiExtern/job-application-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
