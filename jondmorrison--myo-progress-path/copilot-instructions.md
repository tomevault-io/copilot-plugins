## myo-progress-path

> - Sam's Word documents are the authority for all patient-facing content. App content is derived. When content doesn't match the document, the document wins. Always read the relevant section of the Word doc before touching patient-facing content.

# MyoCoach (myo-progress-path)

## Critical Rules
- Sam's Word documents are the authority for all patient-facing content. App content is derived. When content doesn't match the document, the document wins. Always read the relevant section of the Word doc before touching patient-facing content.
- Do not run SeedProgram, syncProgramData, or any seeding scripts — they wipe Vimeo URLs and overwrite exercise data
- Supabase weeks and exercises tables are empty — all program data comes from public/24-week-program.json only
- Vimeo video URLs live directly in public/24-week-program.json — do not look for them in Supabase
- Deploy to production by merging to main — Vercel auto-deploys on every push to main. Branch protection blocks direct pushes; all changes must go through a feature branch + PR + merge
- Read only first, report findings, confirm with Jon, then make changes — never restructure without understanding the full data flow

## Architecture
- Program content: public/24-week-program.json (single source of truth)
- Exercises load from JSON in WeekDetail.tsx — Supabase is fallback only
- Objectives, introduction, tracking, progress_benchmark all load from JSON and merge into week state
- Vimeo URLs are in JSON exercises[].demo_video_url as https://player.vimeo.com/video/[id]
- Images for exercises are comma-separated in demo_video_url alongside or instead of video URLs
- Learn Hub articles are markdown files in public/content/learn/
- Learn Hub article index is public/content/learn/index.json

## Key People
- Sam Raniak (samantha.raniak@hotmail.com) — Myofunctional Therapist, content authority
- Matt Francisco (matt@montrosedentalcentre.com) — Dentist, client

## Stack
- Vite + React + TypeScript
- Supabase (auth + uploads + patient progress only — program data is JSON)
- Vercel (hosting + deployment — auto-deploys on push to main)

## Pathways
- program_variant: 'frenectomy' | 'standard' | 'non_frenectomy'
- frenectomy = surgical pathway (26 weeks including post-op)
- standard = non-surgical pathway (24 weeks)
- non_frenectomy = basic no-feedback pathway (messaging disabled)

## Common Mistakes to Avoid
- Don't change exercises, checklist, demo_video_url, or tracking fields when only updating objectives or introduction
- Don't assume Supabase has data — query it first and check if empty
- Don't use the /images/assets/ path for vu-hickey — correct path is /images/learn/vu-hickey.png
- Don't add "Why Compensations Matter" or "What to Watch For" sections to compensations.md — Sam didn't write them
- The standard variant maps to Non-Frenectomy Program in the curriculum tab, not Frenectomy Program
- Supabase weeks and exercises tables are empty — do not query them for program data
- Week 25 curriculum filter uses <= 25, not <= 24, to include Post Program Review
- NasalUnblockModal has known duplicate text issue — only one set of footer notes should exist

## Outstanding Items from Sam (as of March 21)
- Four Goals photo placement — verify each photo matches exact position in Sam's Word doc
- Elastic hold M2-4 — both photos (elastic-on-tongue.jpg + vu-hickey.png) need visual confirmation
- Belly breathing — Sam says remove a photo; demo_video_url is empty but check description field for image refs
- BOLT score M8 — verify recording spot shows on live site after Lovable push
- Module 7 alternating note text — "Alternate each practice session between this exercise and the other two" — verify it shows in UI
- Non-surgical curriculum — verify Lovable sync resolved this
- Sam's new screenshots not yet reviewed — Jon has them, start new session

---
> Source: [JonDMorrison/myo-progress-path](https://github.com/JonDMorrison/myo-progress-path) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
