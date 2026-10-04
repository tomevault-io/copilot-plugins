## resume

> Critical constraints for the LaTeX resume document


## Resume Requirements

### Hard Constraints
1. **Must always be exactly ONE page** - Never allow the document to bleed onto a second page
2. **QR code positioning** - Must be properly aligned on the right side of the header, not floating in the middle
3. **Header layout** - Contact info centered with QR code positioned to the right of the name
4. **ATS parseability** - Critical contact info and resume content must remain selectable text in the document body, using standard section headings, simple bullets, consistent dates, and minimal graphics. Do not rely on headers, footers, tables, text boxes, icons, QR codes, or images for information an applicant tracking system needs to parse.
5. **Confidentiality** - Do not include sensitive or overly specific employer implementation details, internal architecture, proprietary tooling details, internal project names, or business-sensitive AI/cloud cost details. Keep company work descriptions accurate but appropriately high-level for public job applications.

### Styling Guidelines
- Do NOT use underfull/overfull hbox warnings (badness 10000)
- Minimize whitespace and use compact spacing
- All section content must fit on a single page
- Preserve a clean single-column reading order for core resume content wherever possible

### Layout Constraints
- Current margins: left=0.5in, top=0.3in, right=0.5in, bottom=0.3in
- When making changes, prioritize single-page constraint over visual spacing
- Consider reducing \sectionspace or other spacing if page length increases

### Tools to Use
- Use `latexmk -pdf resume.tex` to compile
- Always check for underfull/overfull hbox warnings: `latexmk -pdf resume.tex 2>&1 | grep -i "underfull"`
- After any edit, verify: no LaTeX warnings AND resume is still one page

---
> Source: [bgolski/resume](https://github.com/bgolski/resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
