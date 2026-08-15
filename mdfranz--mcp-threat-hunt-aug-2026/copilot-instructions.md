## mcp-threat-hunt-aug-2026

> - Use npm to install and execute tools to generates html or PDF in `dist`

# Environment Assumptions
- Use npm to install and execute tools to generates html or PDF in `dist`
- Source content for slides is in `src`

# Git Rules
- Do not create feature branches unless asked
- Do not push, that is for humans

# Slide Generation
- Generate slides based on the tool requested (marp, slidev) 
- Do NOT assumpe
- `src/slides-simple.md` uses `marp: true` frontmatter with `html: true` (needed for the multi-column layout below) and a custom `style:` block (reduced section padding, 16pt tables, `.columns` grid).
- During review cycles (any time you're iterating on content, not producing the final deliverable), generate **HTML**, not PDF. It's the fast path — no headless Chrome/Puppeteer dependency — and still respects the `.columns` grid and all custom styling:
  ```
  npx @marp-team/marp-cli src/slides-simple.md -o dist/slides-simple.html --allow-local-files --html
  ```
- Reserve PDF generation for the final deliverable. PDF (and PNG) export goes through headless Chrome, so it needs both `CHROME_PATH` and the `--html` flag or the columns render as literal `<div>` tags instead of a layout:
  ```
  export CHROME_PATH=~/.cache/puppeteer/chrome/<version>/chrome-linux64/chrome
  npx @marp-team/marp-cli src/slides-simple.md -o dist/slides-simple.pdf --allow-local-files --pdf --html
  ```
- For a multi-column slide, wrap columns in `<div class="columns"><div>...</div><div>...</div></div>` (blank line after each opening tag so markdown-it still parses the bullets/bold inside). Keep bullets short — wrapped lines in a narrow column push later content off the bottom of the slide.

# Themes and Style
- Use simple large 20 point fonts with bullets
- Use monochrome (black and white) styling
- Exception: table text may be set to 16 point (`table, table th, table td { font-size: 16pt; }` in the frontmatter `style:` block) so tables with several columns fit without shrinking the surrounding body text
- Exception: blockquote (`> `) asides may be set to 16 point (`blockquote { font-size: 16pt; }` in the same `style:` block) — at full body size a multi-line `>` aside can run past the bottom of the slide, especially on slides that already carry a diagram or a full bullet list above it. Verify by exporting to PNG (see Diagrams below), not just by page count.

# Diagrams
- Author diagrams as Mermaid `flowchart` blocks inline in the source markdown while drafting.
- Marp does not execute Mermaid JS during static PDF/PNG export — a raw ` ```mermaid ` fence renders as plain code text, not a diagram. Before generating output, pre-render each diagram to SVG and swap the fence for an image:
  1. Extract each mermaid block into its own file under `src/diagrams/<slide-name>.mmd`.
  2. Render to SVG with `@mermaid-js/mermaid-cli` (`mmdc` via `npx`), using a monochrome theme config (white background, black nodes/lines/text) so diagrams match deck style. Requires a headless Chrome — point `PUPPETEER_EXECUTABLE_PATH` / a `-p` puppeteer config at a locally installed Chrome (e.g. one cached by `puppeteer` under `~/.cache/puppeteer`) if none is on `PATH`.
     ```
     npx @mermaid-js/mermaid-cli -i src/diagrams/<name>.mmd -o src/diagrams/<name>.svg -c mermaid-mono.json -b white -p puppeteer-config.json
     ```
  3. Replace the ` ```mermaid ` fence in the slide markdown with `![](diagrams/<name>.svg)` (add `w:NNN` sizing, e.g. `![w:550](...)`, if the default render overflows the slide).
  4. Regenerate the deck (marp needs the same Chrome — set `CHROME_PATH`).
- Layout/overflow checks still need a rendered image, not the HTML review build — HTML content can silently overflow a fixed-size slide with no visual cue. Verify by exporting every slide to PNG (`marp --images png --allow-local-files --html`, same Chrome requirement as PDF) and reviewing them — a PDF page count alone won't catch a diagram that's missing, oversized, or pushing later content off the slide.
- Keep both the `.mmd` source and the rendered `.svg` in `src/diagrams/` so diagrams are re-renderable and reviewable in diffs.

---
> Source: [mdfranz/mcp-threat-hunt-aug-2026](https://github.com/mdfranz/mcp-threat-hunt-aug-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
