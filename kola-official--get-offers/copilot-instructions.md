## get-offers

> Use this repository as an application-campaign workspace generator for resumes, jobs, internships, mentor outreach, and research applications.

# Get Offers Agent Instructions

Use this repository as an application-campaign workspace generator for resumes, jobs, internships, mentor outreach, and research applications.

Before acting, read:

- `skills/get-offers/SKILL.md`
- `skills/get-offers/references/workflow.md`
- `skills/get-offers/references/material-guidelines.md`
- `skills/get-offers/references/support-matrix.md`
- `skills/get-offers/integrations/runtime_support.json`
- `简历参考编写指南（通用岗）.md`
- `简历参考编写指南（技术岗）.md`
- `简历参考编写指南（非技术）.md`
- `简历参考编写指南（求学）.md`

Default behavior:

- Create or improve an initial `resume.md` from user-provided facts.
- Recommend target directions when the user has no target list.
- Preserve target email, URL, platform, salary, source, and notes.
- Generate target-specific materials under `materials/<target>/`.
- Prefer IMAP draft upload or local `.eml` review drafts over immediate sending.
- Do not send email or trigger browser-side application automation unless the user explicitly opts into full-auto mode.

Useful scripts:

- `skills/get-offers/scripts/init_application_project.py`
- `skills/get-offers/scripts/run_campaign_pipeline.py`
- `skills/get-offers/scripts/make_material_drafts.py`
- `skills/get-offers/scripts/fetch_public_jobs.py`
- `skills/get-offers/scripts/collect_faculty_targets.py`
- `skills/get-offers/scripts/deep_read_faculty_profiles.py`
- `skills/get-offers/scripts/export_resume_images.py`
- `skills/get-offers/scripts/make_email_drafts.py`
- `skills/get-offers/scripts/upload_email_drafts.py`
- `skills/get-offers/scripts/send_email_messages.py`

Run `python3 skills/get-offers/scripts/validate_workflow_smoke.py` after changing workflow scripts.

---
> Source: [kola-official/get-offers](https://github.com/kola-official/get-offers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
