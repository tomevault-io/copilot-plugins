## cv

> This is a TypeScript + Astro CV builder. One Markdown file (`src/content/resume/resume.md`) is the

This is a TypeScript + Astro CV builder. One Markdown file (`src/content/resume/resume.md`) is the
single source of truth for all outputs: web page (Astro → GitHub Pages), downloadable PDF
(`@react-pdf/renderer`, no browser), and ATS feedback report (Claude API).

## Session Start Protocol

Read `BOOT.md` (project context, invariants, REVIEW protocol) then `WAL.md` (current project state,
pending work, what NOT to touch) before taking any action.

## WAL Update Protocol

Before ending a session, update `WAL.md`:
1. Collapse completed items to one-line summaries (details belong in git history).
2. Add any REVIEW markers created this session under `## Open REVIEW Items`.
3. Update phase status if any phase changed state.
4. Keep WAL under 3,000 tokens total.

## REVIEW Marker Protocol

When encountering a spec ambiguity, contradiction, or decision requiring author input:
1. Implement the spec as written — do not invent alternatives.
2. Add `<!-- REVIEW: [reason] — spec://cv/relevant#section -->` at the relevant location.
3. Record the item in `WAL.md` under `## Open REVIEW Items`.
4. Report to the author at end of session. Do NOT modify specs unilaterally.

---

## Tech stack

| Concern | Choice |
|---|---|
| Package manager | **pnpm** — always use `pnpm`, never npm or yarn |
| Framework | Astro 5 with TypeScript strict mode throughout |
| Styling | Tailwind CSS v4 — CSS-based `@theme` config in `global.css`, no `tailwind.config.js` |
| PDF | `@react-pdf/renderer` — programmatic, Node-only, no browser/Playwright in CI |
| Fonts | `@fontsource/open-sans` — WOFF from npm, registered in `src/pdf/fonts.ts` |
| ATS check | `@anthropic-ai/sdk` via `scripts/ats-check.ts` |
| Data | Astro Content Collections — YAML front matter in `src/content/resume/resume.md` |
| CI/CD | GitHub Actions (3 workflows: deploy-pages, build-pdf, ats-check) |

## Key file paths

- `BOOT.md` — universal AI entry point; read this first every session
- `WAL.md` — session continuation state; current phase, pending work, REVIEW items
- `src/content/resume/resume.md` — CV data (YAML front matter); source of truth for web + PDF
- `src/content/achievements/achievements.md` — LinkedIn achievements; web-only, never in PDF
- `src/content/config.ts` — Zod schemas for both collections
- `src/pdf/` — React-PDF components (separate visual implementation from Astro components)
- `src/pdf/fonts.ts` — `Font.register()` for Open Sans via `@fontsource`
- `scripts/generate-pdf.tsx` — reads front matter, calls `renderToFile()` → `dist/CV.pdf`
- `scripts/ats-check.ts` — reads resume + JD, calls Claude API, writes `ats-report.md`
- `docs/spec/INDEX.md` — full spec directory

## Architectural constraints

- **Data vs presentation are decoupled**: web (Astro) and PDF (React-PDF) share data, not components.
- **No browser in CI for PDF**: `@react-pdf/renderer` runs as pure Node.js. Playwright is for dev preview only.
- **Tailwind v4**: integrated via `@tailwindcss/vite` Vite plugin in `astro.config.ts`. No `@astrojs/tailwind`.
- **CV layout must be single-column A4**: no tables, no multi-column, no icons in text flow — ATS compatibility.
- **Fonts**: Open Sans via `@fontsource/open-sans`. React-PDF fonts use `path.resolve(process.cwd(), ...)`.
- **`src/content/achievements/`** feeds the `/achievements` web page only — never included in the 1-page PDF.

## AI presence disclosure

spec: spec://cv/security#ai-disclosure

AI tools working in this repository must **never** leave unambiguous indicators of AI involvement
in any artefact visible to humans outside this project, unless the author explicitly approves it
for a specific artefact within the session. This policy exists because public git history is
permanent and globally visible; in repressive jurisdictions such disclosures can expose the author
to legal risk.

**Prohibited** without explicit per-artefact approval: `Co-Authored-By:` trailers naming an AI
model, subject lines or footers stating AI authorship (e.g. "🤖 Generated with GitHub Copilot"),
inline code comments attributing code to an AI tool, README badges or PR body footers disclosing
AI involvement.

**Always permitted** (instructions *for* AI, not disclosures *of* AI): this file and equivalents
(`CLAUDE.md`, `AGENTS.md`), Copilot workspace settings, files in `docs/spec/`, internal technical
documentation.

When uncertain, ask the author before producing the output.

---

## Git commit conventions

spec: conventionalcommits.org/en/v1.0.0

Split large changesets into smaller, logical commits — one concern per commit, based on session
history and context.

**Subject line**: use Conventional Commits format `<type>(<scope>): <description>`. Keep it concise
(shown in `git log --oneline`, GUI panels, PR summaries). Common types: `feat`, `fix`, `refactor`,
`chore`, `docs`, `style`, `test`, `ci`, `build`.

**Body**: required for any non-trivial commit. Explain *why* the change was made and the rationale
behind implementation decisions. Include spec references in reverse DNS notation
(e.g. `conventionalcommits.org`, `docs.astro.build`, `react-pdf.org`). Be verbose when the content
is meaningful — do not truncate reasoning for brevity's sake. Separate subject from body with a
blank line.

---
> Source: [sorokoletovdu/cv](https://github.com/sorokoletovdu/cv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
