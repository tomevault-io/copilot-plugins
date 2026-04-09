## ai-visibility-tracker

> - **ALWAYS check your branch first:** `git branch`

# Project Architecture — READ THIS FIRST BEFORE DOING ANYTHING

## Git Workflow — MANDATORY
- **ALWAYS check your branch first:** `git branch`
- **NEVER work directly on `main`.** Switch to `dev` first: `git checkout dev`
- `main` = stable, deployed code. Only changes via merge from `dev`.
- `dev` = where all development happens (features, fixes, experiments)
- **Commit frequently** on `dev` with clear messages
- **To deploy:** `git checkout main && git merge dev && ./deploy_to_cloud_run.sh && git checkout dev`
- **Before starting any new work session:** `git checkout dev && git pull origin dev`
- **NEVER run `git reset --hard`, `git push --force`, or `git clean -f`** — these have caused data loss before

## Deployment
- This app runs on **Google Cloud Run** — NOT locally
- Live URL: https://dashboard.dasilvaconsulting.com
- Cloud Run containers are **EPHEMERAL** — local disk resets on every container restart
- **Never assume local file changes persist on the live app**
- Always deploy changes with: `./deploy_to_cloud_run.sh`
- gcloud binary is at: `/opt/homebrew/share/google-cloud-sdk/bin/gcloud`
- Always use `export PATH="/opt/homebrew/share/google-cloud-sdk/bin:$PATH"` before gcloud commands

## Data Storage — GCS is the Source of Truth
- Permanent storage is **Google Cloud Storage**: bucket `ai-visibility-reports-dasilva`
- GCS folder structure:
  - `client-data/` — brand configs, personas, keywords
  - `prompt-data/` — prompt CSV files (approved and generated)
  - `test-results/` — results_summary.csv and test_*.json files
  - `reports/` — HTML, PDF, CSV reports
- `startup.sh` downloads from GCS to local disk when the container starts
- `run_report.py` uploads results to GCS after tests complete
- Local disk (`data/`) is only a **temporary working copy** — it will be wiped

## PRIORITY: Per-Client Data Isolation (DO THIS BEFORE ONBOARDING NEW CLIENTS)
**Current architecture is BROKEN:** All clients share one `results_summary.csv` and one `reports/` folder.
This causes cross-client data contamination (e.g., Natasha Denona personas appearing in OCO reports).

**Required fix — implement per-client isolated folders:**
- Local: `data/results/{client_slug}/results_summary.csv`, `data/reports/{client_slug}/`
- GCS: `test-results/{client_slug}/`, `reports/{client_slug}/`

**Files that need changes (7 total):**
1. `main.py` — pass client_slug to ResultsTracker
2. `src/tracking/results_tracker.py` — use `data/results/{client_slug}/`
3. `src/tracking/historical_tracker.py` — use `data/results/{client_slug}/monthly_scores.json`
4. `src/client_manager/gcs_sync.py` — add client_slug to upload/download methods
5. `src/storage/gcs_manager.py` — add client_slug to report methods
6. `dashboard_pages/run_report.py` — pass client_slug through to main.py
7. `dashboard_pages/overview.py` — read from `data/reports/{client_slug}/`

**DO NOT onboard new paying clients until this is implemented.**

## API Keys
- API keys are NOT in the Docker image (config.json and secrets.toml are in .dockerignore)
- Keys live in Streamlit secrets (GCP Secret Manager: `streamlit-secrets`)
- `run_report.py` passes keys as environment variables to the `main.py` subprocess
- `main.py` reads them via `os.getenv()` as a fallback — this is intentional

## Current Active Client
- **Ontario Caregiver Organization (OCO)**
- Brand config: `data/ontario_caregiver_organization/ontario_caregiver_organization_brand_config.json`
- Personas file: `data/ontario_caregiver_organization/ontario_caregiver_organization_personas.json`
- Prompts file: `data/ontario_caregiver_organization/ontario_caregiver_organization_prompts.csv`
- Persona JSON structure: top-level key is `personas` (a list)
- Persona item keys: `id`, `name`, `weight`, `description`, `caregiving_role`, `key_trigger`, `priority_program`, `top_barrier`, `priority_topics`
- **Persona lookup key is `name`** — NOT `target`, NOT `persona`

## Correct Personas for OCO (8 total)
1. Adult Child Caregiver (50+)
2. Adult Child Caregiver (Under 50)
3. Spousal Caregiver
4. Parent of Child w/ Complex Needs
5. Mental Health Caregiver
6. Young Caregiver (Under 26)
7. Healthcare Provider
8. Employer / HR Leader

## Known History — Do Not Repeat These Mistakes
- **DO NOT delete GCS test results** — they are the only permanent record
- Old Natasha Denona results were deleted from GCS in error (March 2026) — they cannot be recovered
- `.dockerignore` has `*.json` wildcard — this is intentional for security, do not remove it
- `csv_exporter.py` hard bracket accesses on `source` dict were fixed to use `.get()` with defaults
- `gap_analyzer.py` builds `prioritized_audiences` dicts with key `'persona'` (a string name)

## Subprocess / Working Directory
- `main.py` is called as a subprocess from `run_report.py` via `subprocess.Popen`
- The subprocess must run with `cwd=` explicitly set to the project root
- Results are written to `data/results/results_summary.csv` relative to cwd
- If cwd is wrong, results write to one path and get read from another → "No test results found"

## Other Clients
- Dripos (coffee shop POS)
- UniUni (Canadian logistics)
- Natasha Denona (beauty — historical, no longer active)

## Do Not Do These Things
- Do not work on the `main` branch — always use `dev`
- Do not run `python main.py` directly to test — always test through the dashboard
- Do not make changes locally and assume they are live — deploy first
- Do not fix "local" data issues — fix GCS data issues
- Do not use `gsutil` without checking it's installed (`which gsutil`)
- Do not use bare `gcloud` — use the full path or export PATH first
- Do not run `git reset --hard`, `git push --force`, or `git clean -f`
- Do not delete anything from GCS without explicit confirmation from Tiffany

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dasilvacon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dasilvacon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
