## ocr

> - Keep responses short, direct, and in bullets when reporting work.

# Agent Instructions

- Keep responses short, direct, and in bullets when reporting work.
- Keep every code file below 500 lines.
- Split code by clear responsibility before a file becomes hard to edit.
- Prefer small modules with focused functions over large mixed-purpose files.
- Add concise comments only where they help the next agent understand intent quickly.
- Do not remove generated-label fields without updating schema, docs, and reconstruction.
- For paired translation columns, preserve semantic color matching where possible.
- Rebuild and smoke-test both standalone exes after GUI or generator changes.
- Keep public non-sensitive generated samples easy to find at
  `docs/dataset/samples/images/`, with matching labels at
  `docs/dataset/samples/labels/`, and document those paths in sample docs.
- Publish order for this project always starts with SaroTek: commit and push the
  completed work to `origin` (`https://github.com/saroltech/OCR.git`) first.
- After SaroTek has the commit, push the same commit to the open-source
  community repo `upstream` (`https://github.com/EOTCOpenSource/OCR.git`) on a
  specific feature branch.
- Merge into the open-source `main` branch only when the user explicitly asks
  for that merge, and only after checking that the branch can fast-forward or
  merge without conflict.

---
> Source: [EOTCOpenSource/OCR](https://github.com/EOTCOpenSource/OCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
