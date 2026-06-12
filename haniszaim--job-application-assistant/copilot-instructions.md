## job-application-assistant

> An automated job application pipeline for a fresh graduate in Malaysia. It:

# job-apply-bot

## What This Project Does

An automated job application pipeline for a fresh graduate in Malaysia. It:

1. Scrapes Google Jobs using Playwright for AI/ML/data roles across Malaysian cities
2. Deduplicates and filters raw job listings
3. Generates tailored cover letters, email drafts, and interview study guides using Claude (Haiku) based on the candidate's CV
4. Shows the generated content in the terminal for human approval before anything is saved or sent
5. On approval: saves `.docx`, `.pdf`, and `.md` files locally; logs the application to Google Sheets; optionally creates a Gmail draft with the cover letter attached

## Tech Stack

| Tool | Version / Notes |
|---|---|
| Python | 3.x (venv in `venv/`) |
| Playwright | Sync API, Chromium, `headless=False` |
| Anthropic SDK | `claude-haiku-4-5-20251001` for all generation |
| gspread | Google Sheets logging via service account |
| Google API client | Gmail draft creation via OAuth2 |
| python-docx | `.docx` cover letter output |
| fpdf2 | PDF cover letter output |
| PyMuPDF (fitz) | Reads candidate CV from PDF |
| python-dotenv | Loads secrets from `.env` |
| PyYAML | Reads `config.yaml` |

## File Structure

```
job-apply-bot/
├── main.py                  # Entry point; orchestrates the full pipeline
├── config.yaml              # Candidate name, CV path, target roles & locations
├── .env                     # Secrets (not committed): ANTHROPIC_API_KEY, SHEET_ID
├── cv/
│   └── HANISZAIM_RESUME.pdf # Candidate's CV — read at generation time
├── agents/
│   ├── scraper.py           # Google Jobs scraper using Playwright
│   ├── scorer.py            # Deduplication and listing-page filtering
│   ├── generator.py         # Claude API calls to produce cover letter, email, study guide
│   ├── approval.py          # Terminal review UI (approve / edit / reject)
│   ├── file_saver.py        # Saves .docx, .pdf, study guide .md to output/
│   ├── tracker.py           # Logs application row to Google Sheets
│   └── gmail.py             # Creates Gmail draft with PDF attachment
├── prompts/
│   ├── cover_letter.txt     # Prompt template with {cv_content}, {company_name}, etc.
│   ├── email_draft.txt      # Short email prompt template
│   └── study_guide.txt      # Interview prep prompt template
├── credentials/
│   ├── google_credentials.json   # Service account key for Sheets (not committed)
│   └── gmail_credentials.json    # OAuth2 client credentials for Gmail (not committed)
└── output/
    └── {Company}_{Role}_{date}/  # Created per approved application
        ├── cover_letter.docx
        ├── cover_letter.pdf
        └── study_guide.md
```

## Key Conventions

### Pipeline flow
`scrape_all()` → `filter_jobs()` → `run_pipeline()` per job. The pipeline is synchronous and interactive — the user approves each job before anything is written or sent.

### Config-driven search
Roles and locations live entirely in `config.yaml`. Do not hardcode them in agent files. The scraper deduplicates roles with a `seen` set at load time to handle duplicates in the config.

### Human-like scraping
The scraper adds random delays (`human_delay`) between page loads and card clicks to avoid CAPTCHA detection. Keep these delays. The browser runs `headless=False` intentionally so the user can monitor it.

### Prompt templates
Prompts are plain `.txt` files with Python `str.format()` placeholders (`{cv_content}`, `{company_name}`, etc.). Do not use f-strings in the prompt files. Keep prompt logic in `prompts/` and formatting logic in `generator.py`.

### Claude model in use
All three generation calls use `claude-haiku-4-5-20251001`. Do not swap to Sonnet or Opus without checking cost — this runs once per job in a potentially large batch.

### Output folder naming
`{company}_{role}_{date.today()}` with spaces replaced by `_`. Do not change this format — it is also the value logged to Google Sheets.

### Google Sheets row order
`[date, company, role, url, source, location, cover_letter_path, cv_version, "", status, "", ""]` — 12 columns. The three empty strings are reserved columns. Preserve order if adding columns.

### Approval flow return values
`approval.review()` returns either the string `"approved"`, the string `"rejected"`, or the tuple `("edit", note_string)`. `main.py` checks all three branches. The edit branch is a stub — do not silently drop it.

## Always Do

- Load all secrets from `.env` via `python-dotenv` — never hardcode API keys or sheet IDs
- Preserve `human_delay()` calls in the scraper — removing them risks CAPTCHA blocks
- Respect the human approval gate: never auto-approve or skip `approval.review()`
- Use `Path(__file__).parent.parent` to resolve `BASE_DIR` in agents so the project works regardless of working directory
- Keep prompts in `prompts/` as `.txt` template files — do not embed long prompts as strings in Python files
- Deduplicate roles at config load time (already done in `scraper.py`) when the roles list in config grows

## Never Do

- Do not commit `.env`, `credentials/google_credentials.json`, or `credentials/gmail_token.json` — they contain live secrets
- Do not run the scraper `headless=True` without testing CAPTCHA behaviour first
- Do not call `generate()` before `filter_jobs()` — the deduplication step exists to avoid burning API tokens on duplicate listings
- Do not change the Google Sheets row column order without updating `tracker.log_application()` and the sheet header simultaneously
- Do not write cover letter content directly to files without going through `file_saver.save_files()` — it handles both `.docx` and `.pdf` output together
- Do not use `arial.ttf` font path (`C:/Windows/Fonts/arial.ttf`) on non-Windows machines — flag this if porting to Linux/macOS

---
> Source: [haniszaim/Job-Application-Assistant-](https://github.com/haniszaim/Job-Application-Assistant-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
