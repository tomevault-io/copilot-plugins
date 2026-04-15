## tserre-github-io

> WCAG 2.1 AA accessibility compliance (Brown University policy)


# Accessibility: WCAG 2.1 AA Compliance (Brown University Policy)

All HTML output from this project must comply with WCAG 2.1 AA.
Brown University requires full digital accessibility compliance by May 2026.
Reference: https://digital-accessibility.brown.edu/

**Scope:** Applies to all pages including the CPSY 1950 course site. Do not introduce accessibility regressions when editing course content, layouts, or navigation.

## Known Issues to Fix

### CRITICAL: Publications page heading hierarchy
Every publication title is rendered as <h1>. There must be exactly ONE <h1> per page
("Publications"). Year headings should be <h2>. Paper titles should be <h3>.

### Research page heading skip
Page jumps from <h1>Research</h1> directly to <h3> for research themes.
Add <h2> section headers (e.g., "Research Themes") or change themes to <h2>.

### Footer heading levels
Footer uses <h3> then <h4> which may not follow the page's heading flow.
Consider using styled <div> or <span> elements instead of heading tags in the
footer, since the <nav aria-label="..."> landmarks already convey structure.

## Mandatory Checks for Every Change

### Headings
- Single <h1> per page/route
- Headings nest logically: h1 > h2 > h3, never skip levels
- Publication titles are NEVER <h1> — use <h3> under year <h2> headings

### Structure
- Landmark elements present: <header>, <nav>, <main>, <footer>
- All <nav> elements have aria-label to distinguish them
- Skip-to-content link is the first focusable element
- lang="en-US" on <html> element

### Images & Media
- Every <img> has alt text (descriptive for meaningful, alt="" for decorative)
- Decorative images also get aria-hidden="true"
- No auto-playing audio or video without user controls
- <iframe> elements (embedded videos) must have a title attribute

### Color & Contrast
- Text-to-background contrast ratio >= 4.5:1 (>= 3:1 for large text)
- Information is never conveyed by color alone

### Interactive Elements
- All interactive elements reachable via keyboard
- Visible :focus-visible styles on all focusable elements
- Icon-only buttons/links have aria-label or visually-hidden text

### Links
- Link text is descriptive (no bare "click here")
- External links indicate this via aria-label (e.g., "opens in new tab")

### SPA Behavior
- Route changes update document.title
- Route changes announce new content via aria-live region
- Focus moves to new content heading after navigation

### PDFs
- Do not host local PDFs unless they are tagged (PDF/UA compliant)
- Prefer linking to publisher DOI for publications
- Any locally hosted PDF must have title, author, and lang metadata

## Testing
- Run Lighthouse accessibility audit before each deploy
- Test keyboard navigation (Tab, Shift+Tab, Enter, Escape)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tserre)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tserre)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
