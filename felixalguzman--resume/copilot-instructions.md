## resume

> Sources for Félix Guzmán's resume plus the PDF render pipeline.

# Resume repo

Sources for Félix Guzmán's resume plus the PDF render pipeline.

## Files

- `resume.md` — canonical content. Edit this first.
- `resume.html` — styled version actually rendered to PDF. Every content change in `resume.md` must be mirrored here manually (no md→html build step).
- `render_resume.sh` — renders `resume.html` to `Felix_Guzman_Resume.pdf` via headless Chromium.
- `Felix_Guzman_Resume.pdf` — generated output, committed.

## Workflow

1. Edit `resume.md` and `resume.html` together (same wording).
2. Run `./render_resume.sh`.
3. Check `pdfinfo Felix_Guzman_Resume.pdf | grep Pages` — **must be 1 page**. If it spills, tighten CSS spacing (line-height, section margins, paddings) in `resume.html`, not content.
4. Visual check: `pdftoppm -png -r 60 Felix_Guzman_Resume.pdf /tmp/resume_page` and view the images.

## Content rules (owner decisions)

- No fabricated or top-of-head metrics (e.g. removed an unverifiable "50% efficiency" claim). Only claims Félix can defend in an interview.
- Do NOT list Vue work at AvatharTech — it was AI-generated, owner doesn't want to claim it.
- Do NOT list freelance/client side work (e.g. victoria-lparck).
- Verified timeline (Odoo cert signed 2023-07-31 = Alterna exit year; git history of Clickping repos): Alterna 2018 – mid-2023, Clickping Dec 2023 – Jan 2025, AvatharTech Apr 2025 – present. (Post-2025 commits in siniestros under Félix's email are just dependabot PR merges, not maintenance — owner does not maintain it since leaving.) Resume shows year-only: 2018–2023, 2023–2025, Apr 2025–present. The Alterna utility-billing app tailed into 2024 as wrap-up work; don't put that year range on the bullet (creates apparent inconsistency with section dates).
- Fill-Click (Alterna) architecture: React frontend, ASP.NET Core web backend, Spring Boot service ingesting machine/HMI data over MQTT, PostgreSQL, plus a Flutter operator mobile app. The Grails/PostgreSQL version was only the 2018 internship prototype (see "Reporte de pasantía.pdf" in ~/Downloads).
- Projects section: Watch Insights (private GitLab repo — don't link) and Constellation (public: gitlab.com/felixalguzman/constellation). No GitLab profile link in the header — profile is cluttered with old university repos; GitHub link suffices.
- Titles (grilled 2026-07): Alterna = "Lead Developer" (sole dev first ~2 years, led architecture as team grew to 4-5). Clickping = "Software Developer" — small shop, no formal titles, don't claim "Senior".
- No Frappe in skills — real experience exists but owner can't/won't discuss it in interviews.
- Watch Insights privacy angle is intentional and verified: vendor denied external uploads; jadx decompile of their SDK shows hardcoded ECG upload endpoint (docs/reverse/PROTOCOL_MAP.md in the repo).
- Clickping and AvatharTech had more apps than listed — listed ones are the deliberate highlights.
- No AvatharTech internal product/client names on the resume (Kepler, NimboERP, Avasure) and no DGII/e-CF mention — describe by function only ("core ERP", "insurance quoting platform"). Fill-Click and ITRS are fine to name.

## Source repos for bullet material

- AvatharTech work: github.com/AvatharTech (site-procigar, control-ingreso, control-ingreso-movil).
- Clickping work: `~/Documents/GitHub/siniestros`, `~/Documents/GitHub/itrs`, `~/Documents/GitHub/itrs_telegram_bot` (Global-B org).
- Personal projects: gitlab.com/felixalguzman (watch_insights, constellation, dubarr).

---
> Source: [felixalguzman/resume](https://github.com/felixalguzman/resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
