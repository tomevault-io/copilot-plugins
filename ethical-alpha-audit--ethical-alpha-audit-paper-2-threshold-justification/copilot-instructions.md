## prior-portfolio-manuscript

> Manuscript handling discipline (.docx files)


# Manuscript handling

Manuscripts in this repository are stored as `.docx` files in `inputs/manuscript.docx` (primary canonical version) and optionally `inputs/supplementary.docx`.

When working on manuscripts:

- **Do not edit .docx files directly.** Convert to markdown via `pandoc inputs/manuscript.docx -o /tmp/manuscript.md`, edit the markdown, then convert back if needed. The markdown is the working format; the .docx is the deliverable.
- **Never reformat or "clean up" tracked changes** without explicit user instruction. Tracked changes are part of the editorial audit trail.
- **Forbidden tokens.** The following tokens must not appear in any committed manuscript: `_FLAGGED`, `sweet spot`, `sweet-spot`, `152-fold`, `PhysioNet` (unless citing it), `policy-proportionate`, `five-paper`. The reliable check extracts text from the .docx via zipfile and scans:

  ```
  python -c "
  import zipfile, re
  with zipfile.ZipFile('inputs/manuscript.docx') as z:
      text = re.sub(r'<[^>]+>', '', z.read('word/document.xml').decode('utf-8','ignore'))
  forbidden = ['_FLAGGED', 'sweet spot', 'sweet-spot', '152-fold', 'policy-proportionate', 'five-paper']
  for t in forbidden:
      if t in text: print(f'FAIL: {t}'); exit(1)
  print('PASS')
  "
  ```

- **Version filenames use hyphens, not dots.** `manuscript_v2-1.docx`, not `manuscript_v2.1.docx`. Dots in filenames trigger MIME-type parsing failures in some iOS apps.

---
> Source: [ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification](https://github.com/ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
