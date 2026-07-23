## private-gpt

> See @README.md, @docs/SOURCE_OF_TRUTH.md, @docs/PRD.md, and @docs/STYLE_GUIDE.md before editing `index.html`.

# Claude Code Instructions

See @README.md, @docs/SOURCE_OF_TRUTH.md, @docs/PRD.md, and @docs/STYLE_GUIDE.md before editing `index.html`.

## Scope

- Keep the app static and easy to inspect.
- Keep `index.html` as the only runtime implementation file unless explicitly told otherwise.
- Treat `docs/SOURCE_OF_TRUTH.md` as the canonical pointer to API contract paths and document ownership.
- Update the relevant docs in `docs/` whenever behavior, visuals, persistence, security posture, or API request/response handling changes.

## Claude-Specific Notes

- Use the shared docs above as the source of truth instead of duplicating product or design rules here.
- If implementation and docs disagree, fix the disagreement in the same change.
- Validate `index.html` changes with the documented script and manually test changed flows when needed.

---
> Source: [zylon-ai/private-gpt](https://github.com/zylon-ai/private-gpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
