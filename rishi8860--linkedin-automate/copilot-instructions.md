## linkedin-automate

> Playwright + Gemini automation that searches LinkedIn jobs and completes Easy Apply forms.

# LinkedInApply

Playwright + Gemini automation that searches LinkedIn jobs and completes Easy Apply forms.

## Setup

```powershell
.\venv\Scripts\python.exe -m pip install -r requirements.txt
.\venv\Scripts\python.exe -m playwright install chromium
```

Put a real key in `.env` (copy from `.env.example`):

```
GEMINI_API_KEY=...
```

## Run order

```powershell
# 1. One-time: log in and cache the session to linkedin_state.json
.\venv\Scripts\python.exe linkedin_login.py

# 2. One-time (optional): parse the resume into resume_profile.json
.\venv\Scripts\python.exe resume_profile.py --force

# 3. Search + Easy Apply
.\venv\Scripts\python.exe linkedin_search.py
.\venv\Scripts\python.exe linkedin_search.py "Data Scientist" 50 --max 3
```

Step 2 runs automatically as part of step 3 if `resume_profile.json` is missing.

## Files

| File | Purpose |
| --- | --- |
| `config.json` | Role, applicant cap, resume path, max applications, model |
| `linkedin_login.py` | Headed login, saves `linkedin_state.json` |
| `resume_profile.py` | Resume -> `resume_profile.json` via Gemini (cached) |
| `qa_store.py` | Question/answer cache in `qa_cache.json` |
| `easy_apply.py` | Reads modal fields, answers via Gemini, submits |
| `linkedin_search.py` | Orchestrator: search -> iterate cards -> apply |
| `gemini_client.py` | Gemini API wrapper |
| `dump_page.py` | Debug tool: dumps HTML + probes selectors when scraping breaks |

## IMPORTANT: use `/jobs/search/`, never the search bar

LinkedIn has two job-results pages:

- `/jobs/search/` — **classic DOM.** Cards are `li[data-occludable-job-id]`
  (25 per page), titles at `a.job-card-list__title--link`,
  `button.jobs-apply-button` present. This is what the scripts use.
- `/jobs/search-results/` — **server-driven UI.** Cards are `div[role="button"]`
  with hashed `componentkey` attributes, no job id, no anchor. Nothing scrapeable.

Typing into the global search bar routes to the SDUI page, which is why an
earlier version reported `0 job card(s)`. `open_jobs_search()` therefore builds a
`/jobs/search/` URL directly and re-navigates if LinkedIn redirects.

Cards are **occluded**: only ~7 of 25 are hydrated at a time. `hydrate_cards()`
scrolls the list container to force rendering, and each card is scrolled into
view again before clicking.

When selectors break, run `dump_page.py` — it writes `debug_page.html` and prints
counts for every candidate selector plus the repeated list shapes on the page.

## Behaviour notes

- `linkedin_search.py` **auto-submits** applications. Set `max_applications` low while testing.
- `keywords_mode` defaults to `"role"`: only the role is sent as keywords, because
  LinkedIn treats the full template ("hiring under 100 applicants posted in the
  past 24 hours") as literal search terms and buries good results. Recency and
  Easy Apply are enforced by the `f_TPR=r86400` / `f_AL=true` URL filters, and the
  applicant cap is enforced by reading the count off the details panel and
  skipping jobs above `applicants`. Set to `"template"` to send the full string.
- Answers are cached in `qa_cache.json` by normalised question text, so repeat
  questions never hit the API twice. A cached answer is discarded if it is no
  longer one of the offered options.
- LinkedIn's job detail panel uses hashed class names. The Easy Apply button is
  located by ARIA role/name, not by class. Form fields are discovered by tagging
  every control in the modal with `data-aa-idx` via `COLLECT_FIELDS_JS`.
- The "Follow <Company>" checkbox is always unchecked.
- Numeric fields (detected by `type=number` or a "how many / years of" label) get
  the answer reduced to a bare number. `tel` fields keep their formatting.
- Screenshots are written on failure: `error_apply_N.png`,
  `easy_apply_stuck_stepN.png`, `easy_apply_step_limit.png`.

## Verification

No test framework is set up. To sanity-check changes:

```powershell
.\venv\Scripts\python.exe -m py_compile linkedin_search.py easy_apply.py qa_store.py resume_profile.py gemini_client.py linkedin_login.py
```

Field detection and filling can be exercised against a mock modal with
`page.set_content(...)` plus `collect_fields` / `fill_field` — no LinkedIn or
Gemini calls needed.

---
> Source: [Rishi8860/LinkedIn-Automate](https://github.com/Rishi8860/LinkedIn-Automate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
