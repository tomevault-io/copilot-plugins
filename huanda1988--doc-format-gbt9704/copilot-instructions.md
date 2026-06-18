## doc-format-gbt9704

> Format Chinese official documents according to GB/T 9704-2012 and the local AI rule file for 党政机关公文格式. Use when the user asks to排版/规范化/校验/转换 Word DOC/DOCX/RTF/TXT documents as 党政机关公文、公文格式、GB/T 9704、GBT9704、9704-2012, including applying A4版心、3号仿宋、2号小标宋标题、层级序号、页码、署名日期、版记/附件/信函/命令/纪要 rules.


# GB/T 9704 Official Document Formatting

Use this skill to format Chinese official documents according to GB/T 9704-2012《党政机关公文格式》. For DOC/DOCX work, use the Documents skill/tooling when available and follow its render-to-PNG visual QA requirement before delivery.

## Core Workflow

1. Identify the source document type: `.doc`, `.docx`, `.rtf`, `.txt`, pasted text, or a newly drafted document.
2. If the task touches Word/DOCX files, load/use the `documents:documents` skill and workspace dependency runtimes.
3. Convert legacy `.doc` to `.docx` before editing. Prefer bundled LibreOffice; if it fails, use macOS `textutil` as a fallback and preserve text order.
4. Read `references/format_rules.md` when exact GB/T 9704 parameters or element placement rules are needed.
5. Preserve the source content unless the user explicitly requests rewriting. Only normalize obvious structure issues required for official-document numbering, such as restoring a missing first-level `一、` when later sections contain `二、` and `三、`.
6. Apply page geometry, fonts, paragraph rhythm, title hierarchy, signature/date, page numbers, attachments, and imprint rules.
7. Render the final DOCX to PNG pages with the Documents renderer and inspect every page. Fix clipping, overlap, missing glyphs, bad page numbers, bad signature placement, or awkward page breaks before delivery.
8. Deliver only the final DOCX unless the user asks for QA images or PDF intermediates.

## Quick DOCX Formatting Script

For straightforward reports/summaries without seals, attachments, or imprint blocks, use:

```bash
python scripts/format_docx_gbt9704.py input.docx --out output.docx
```

The script applies:

- A4 page: 210 mm × 297 mm.
- Margins: top 37 mm, left 28 mm, right 26 mm, bottom 35 mm.
- Default body: 3号仿宋, exact 29 pt line pitch, first-line indent 2 Chinese characters.
- Title: 2号小标宋-style font, centered.
- Section hierarchy: first level 黑体, second level 楷体, third/fourth level 仿宋.
- Page number footer: 4号宋体-style `— PAGE —`, odd pages right, even pages left.
- Final issuing authority and date aligned to the right for no-seal documents.

Read or patch the script when the source has special requirements such as seals, attachments, imprint blocks, letter format, command format, or minutes format.

## Manual Formatting Rules

When script defaults are insufficient, apply these high-priority rules:

- Page and版心 must match GB/T 9704: A4, top 37 mm, left 28 mm, right 26 mm, bottom 35 mm, text block 156 mm × 225 mm.
- Use 3号仿宋 for unspecified elements, black text, 22 lines per page, 28 Chinese characters per line as the target layout.
- Use 2号小标宋 for the title, centered below any red separator/header area.
- Body paragraphs start with two-character indentation and wrap flush to the left edge.
- Numbering hierarchy is `一、`, `（一）`, `1.`, `（1）`; generally style level 1 as 黑体, level 2 as 楷体, levels 3-4 as 仿宋.
- Dates use Arabic numerals with full year and no leading zero in month/day: `2019年9月12日`.
- Page numbers sit outside the text block below the版心, with one-character dash lines on both sides; odd pages right, even pages left.
- If the first page would not show正文 because recipients are too long, move recipients into the imprint.
- Do not add punctuation after attachment names in attachment descriptions.

## Verification Checklist

Before final response:

- Confirm output file exists and opens/renders.
- Confirm page geometry and margins programmatically if possible.
- Inspect rendered PNGs page by page at 100% zoom.
- Confirm no text overlaps, clipping, missing glyph boxes, broken tables, or page-number misplacement.
- Confirm final signature/date are visible and aligned appropriately.
- Mention any fallback used, especially if rendering could not be completed.

## Resources

- `references/format_rules.md`: concise GB/T 9704-2012 rule reference for AI/document automation.
- `scripts/format_docx_gbt9704.py`: reusable DOCX formatter for common no-seal official documents.

---
> Source: [huanda1988/doc-format-gbt9704](https://github.com/huanda1988/doc-format-gbt9704) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
