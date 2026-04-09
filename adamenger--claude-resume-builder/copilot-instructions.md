## claude-resume-builder

> You are helping the user build a professional resume as a styled HTML page that converts to PDF.

# Claude Resume Builder

You are helping the user build a professional resume as a styled HTML page that converts to PDF.

## Getting Started

1. Check if `resume.txt` or `resume_old.txt` exists — use it as the starting point for content
2. If no resume file exists, ask the user for their information interactively:
   - Name, contact info, links
   - Professional summary
   - Skills (grouped by category)
   - Work experience (company, title, dates, bullet points)
   - Notable projects or highlights
   - Recognition, certifications, publications
3. Generate `resume.html` using the template structure below and theme values from `config.yaml`

## Theme

Read `config.yaml` for all theme values. When generating or updating `resume.html`:
- Use CSS custom properties (`--bg`, `--text`, `--muted`, `--accent`, `--border`, `--code-bg`, `--bright`, `--font`)
- Set the `@import url()` from `theme.font_import`
- Set body padding from `theme.padding`
- The `:root` block in the HTML should match config.yaml values

Users customize their theme by editing `config.yaml`, not the CSS directly.

## HTML Template Structure

The resume uses this section order:
1. **Header** — name + contact links (`.header`, `.contact`)
2. **Profile** — professional summary paragraph (`.profile`)
3. **Skills** — categorized skill rows (`.skills-grid`, `.skill-row`)
4. **Highlights** — notable work with optional tags (`.highlight-block`)
5. **Experience** — job history with bullet points (`.job`, `.job-header`)
6. **Recognition** — awards, publications, community (`.recognition-list`)
7. **Footer** — "Built with Claude Resume Builder" link (`.footer`)

Each section uses an `<h2>` with `::before { content: "> " }` for the terminal prompt aesthetic.

Always include a footer at the bottom of the resume:
```html
<div class="footer">
  Built with <a href="https://github.com/adamenger/claude-resume-builder">Claude Resume Builder</a>
</div>
```
Style it with muted text, centered, `font-size: 7.5pt`, with a `border-top` divider.

## Available Commands

- `make build` — apply theme + generate PDF via headless Chrome
- `make test` — run WCAG accessibility check + ATS parsability test
- `make open` — build and open the PDF
- `make clean` — remove generated PDF

## Compliance Requirements

### WCAG Accessibility
- All text must have sufficient color contrast against backgrounds
- Links must be distinguishable
- Semantic HTML structure (h1, h2, h3, ul, li)
- The page must pass `npx pa11y resume.html`

### ATS (Applicant Tracking System) Parsability
- Text must be extractable from the PDF (no images of text)
- Must contain: email, phone number, URLs, job titles, date ranges
- The PDF must pass `python3 test_ats.py`

## Workflow

When the user asks to update their resume:
1. Read the current `resume.html` to understand existing content
2. Make the requested changes
3. Run `make build` to generate the PDF
4. Run `make test` to verify WCAG + ATS compliance
5. If tests fail, fix issues and rebuild

When the user asks to start fresh:
1. Gather their information (from file or interactively)
2. Generate `resume.html` following the template structure
3. Build and test with `make build && make test`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamenger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adamenger)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
