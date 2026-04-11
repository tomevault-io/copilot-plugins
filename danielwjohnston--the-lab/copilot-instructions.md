## the-lab

> The Lab is a full-stack personal dashboard and productivity hub built with:

# Project Instructions for Gemini Code Assist

## Architecture & Stack
The Lab is a full-stack personal dashboard and productivity hub built with:

- **Frontend:** React 19 + Vite + Tailwind CSS 4 + Radix UI + wouter (routing) + tRPC client
- **Backend:** Express + tRPC + Drizzle ORM
- **Database:** MySQL (via mysql2) on port 3306 (`thelab` database)
- **Package manager:** pnpm
- **Tests:** Vitest

### Key Commands
```bash
pnpm dev          # Start dev server (tsx watch)
pnpm build        # Build client (Vite) + server (esbuild)
pnpm start        # Run production build
pnpm check        # TypeScript check (no emit)
pnpm test         # Run vitest
pnpm db:push      # Generate + run Drizzle migrations
```

## Key Infrastructure
- **kwes-web VM:** `10.167.110.110` (Ubuntu 24.04). Runs Nginx reverse proxy, Gitea, Guacamole, Dropshare, Bookclub, and the-lab.
  - **SSH Access:** `ssh kwes-web` (IdentityFile `~/.ssh/kwes-tech-mbp1`)
  - **the-lab API:** Runs on port 3001. Auth bypass available at `/api/dev-login?confirm=1`.
- **Gitea:** `http://10.167.110.110/git/` (SSH port 2222). API Token at `~/.config/gitea/token`.
- **GitHub:** `https://github.com/danielwjohnston`
- **Netlify:** `https://app.netlify.com/teams/newswest9/projects` (Auto-deploy on push to GitHub for select apps).

## Agent Rules: Dan's Dev Diary (tripleD)
The shared append-only work log is located at `dandevdiary.md` in the repo root.

1. **Start of Session:** Read tripleD to understand recent context, decisions, and what's in progress.
2. **During/End of Task:** Append to tripleD immediately after completed tasks with what was done, decisions made, blockers hit, and what's next.
3. **Immutable:** Never delete or rewrite previous entries. Append only.
4. **Format:** `## YYYY-MM-DD — Gemini Code Assist` header (once per session), followed by bullet points.
5. **Cross-reference:** Note file paths if work spans multiple projects.

## Security & Sensitive Files
- **Public Repos (GitHub / the-lab):** Never commit sensitive files, tokens, or IPs.
- **Internal Repos (kwes-foundry / Gitea):** Safe for VM IPs, credentials, API keys, and network diagrams. If a sensitive file is found in `the-lab`, move it to `kwes-foundry`.

## Design Systems
1. **Bauhaus Gold:** Speed of Trust / bookclub. Gold `#C9A84C` on `#0A0A0A`. (Bebas Neue + Work Sans)
2. **Bauhaus Lime Noir:** KWES scripts, emails, general IT. Lime `#BFFF00` on `#0D0D0D`. (Proxima Nova / Gotham / Segoe UI)
3. **Crimson Strength:** StrengthsFinder 2.0. Crimson `#C32F2E` on cream `#F0F1EB`. (Playfair Display + Source Sans 3)

---

## Collated Workflows

### 1. Daily Start (`daily-start`)
1. **Read tripleD** for context.
2. **Check Git statuses** and pull latest changes across project directories.
3. **Identify sensitive files** (ensure none leak into public tracking).
4. **Run Infrastructure Checks:**
   - SSH to `kwes-web` and check `gitea-webhook`.
   - Verify Netlify sites return HTTP 200/302.
   - Verify `kwes-web` sub-paths (bookclub, the-lab, dropshare, git).
5. **Set priorities** for the session based on blockers.

### 2. End-of-Day Wrap (`eod` / `eod-wrap`)
1. **Update tripleD** with today's completed work and decisions.
2. **Git Review & Commit:** Commit changes with descriptive messages in all touched repos.
3. **Multi-Destination Sync:** Push to Gitea (`gitea main`) and GitHub (`origin main`).
4. **Verify Live Deploys:** Do not assume a git push equals a live deployment. Run `curl` checks on Netlify and `kwes-web`.
5. **Sync Dev Diary to Portal:**
   ```bash
   scp dandevdiary.md kwes-web:/tmp/ && ssh kwes-web "sudo cp /tmp/dandevdiary.md /var/www/kwes-web/portal/diary/dandevdiary.md && sudo chown gitea:www-data /var/www/kwes-web/portal/diary/dandevdiary.md"
   ```
6. **Update Main Index & Auth Coverage:** Test the portal pages and ensure internal app auth works.
7. **Update llm-history:** Run `cd ~/llm-history && python3 index.py && python3 summarize.py`.
8. **Send EOD Email:** To Andy (apigg@newswest9.com) and Ryan (lrthompson@newswest9.com), cc'ing dwjohnston. Must be sent via TEGNA SMTP relay (`relay.tgna.tegna.com` port 25) using Python `smtplib`.

### 3. Deploy Sync (`deploy-sync`)
Quick workflow to strictly push and verify deployments:
1. Check unpushed changes.
2. Push to `gitea` and `origin`.
3. Verify Gitea webhook auto-deploys via logs on `kwes-web`.
4. Loop through live URLs on `kwes-web` and Netlify to verify 200 HTTP statuses.
5. Re-sync `dandevdiary.md` to the portal.

### 4. StrengthsFinder Chapter Generation (`strengthsfinder-chapter`)
1. **Source:** `bookclub/strengthsfinder2.0/.../transcripts/`
2. **Create Dir:** `ch-{track}-{theme-slug}/`
3. **Generate 3 Formats:**
   - **HTML:** Full web treatment with Crimson Strength design system.
   - **Markdown:** Clean formatting for Gitea/GitHub.
   - **DOCX:** Printable document with proper font fallbacks.
4. **QA & Deploy:** Review independently, commit to Gitea `strengthsfinder2.git`, and rsync to `kwes-web`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielwjohnston)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielwjohnston)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
