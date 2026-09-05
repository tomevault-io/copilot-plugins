## stjohnbiocare

> Guidance for Claude Code when working in this repo. **These instructions override

# CLAUDE.md — stjohnbiocare.com

Guidance for Claude Code when working in this repo. **These instructions override
default behavior — follow them exactly.**

## What this is

`stjohnbiocare.com` — a warm, human, patient-first website for St. John Biocare &
Dialysis Center Foundation and its sister foundations (Core Renal Center, St. Josef
Renal Care). Astro 5 static site, plain scoped CSS (no Tailwind), deployed to GitHub
Pages via GitHub Actions on every push to `main`.

## Working conventions

- **Sync before editing.** Run `git fetch origin main && git merge --ff-only origin/main`
  at the start of a session before making changes — other sessions may have pushed work,
  and a stale checkout leads to editing code that's already been superseded.
- After a sync that changes `package.json`, run `npm install` before `npm run build`.
- Build with `npm run build` and confirm it passes before committing.

## Careers page (`src/pages/careers.astro`)

- **Every job listing card MUST have a unique `id` anchor** so each opening can be linked
  to directly (e.g. `https://stjohnbiocare.com/careers/#dialysis-technician`). This is a
  standing requirement — never post a listing without one.
  - The `id` is the **kebab-case slug of the role title** (e.g. `Cashier / Front Desk
    Officer` → `id="cashier"`, `Utility Aide` → `id="utility-aide"`, `Dialysis Technician`
    → `id="dialysis-technician"`).
  - Always pair it with `scroll-margin-top:90px` in the card's inline `style` so the
    sticky header doesn't cover the title when the anchor is opened. Pattern:
    `<div id="<role-slug>" class="card" style="margin-top:20px;scroll-margin-top:90px">`.
  - When adding a new listing, also add the role to the application form's
    **"Position applying for"** `<select>` dropdown.
- Clinical-role qualifications and duties come from `docs/psn-hd-personnel-guidelines.md`
  (the PSN recommendations). House requirements (e.g. "no derogatory record", NBI/Police
  clearance) are St. John Biocare additions layered on top — keep them clearly ours.

## Claims & compliance

- **ISO status: NOT YET CERTIFIED (pursuing ISO 9001 in future).** The mission and values
  are deliberately *aligned* with ISO 9001 principles (commitment to meeting applicable
  standards + continual improvement) but must **never state or imply certification**. Do
  NOT add "ISO 9001 certified", a certificate number, a certifying body, or any ISO badge
  anywhere until Dr. Rivero explicitly confirms certification has been achieved. A false
  certification claim is a serious compliance/legal risk.
- **No "diagnostics" service line.** The foundation does not run a diagnostics facility.
  Lab work is described as "lab coordination" with partner laboratories — never
  "diagnostics". Keep the site free of the word "diagnostic(s)".

---
> Source: [williamrivero/stjohnbiocare](https://github.com/williamrivero/stjohnbiocare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
