## smart-file-organizer

> Rules for working in this repository.

# smart-file-organizer

Rules for working in this repository.

- `organizer/classify.py` is where file-type detection lives. Today it's a
  naive extension-string lookup (`EXTENSION_MAP`). See `jira_ticket.md` for
  the fix this repo needs: content-based detection that doesn't blindly
  trust the extension, plus a `_needs_review/` path for anything it can't
  confidently identify.
- `organizer/mover.py` does the actual file moves and keeps `.organize_log.json`
  so a run can be undone. Don't remove the undo path -- it's what makes
  repeated rehearsal on the same `demo-inbox/` folder safe.
- `demo-inbox/` is the folder the Streamlit app organizes by default. Its
  starting contents are restored from `fixtures-pristine/` by
  `reset_demo.sh` -- don't hand-edit `demo-inbox/` expecting it to persist;
  regenerate fixtures via `scripts/generate_fixtures.py` instead if the
  fixture set itself needs to change.
- `fixtures-pristine/` is fixture data for a live demo. Don't "fix" the
  tricky files in there (the misleading extension, the missing extension,
  the unrecognizable one) -- those are deliberate.
- Never widen the ticket's scope. It asks for content-based detection and a
  `_needs_review/` path -- nothing about video/audio deep inspection, ML
  classifiers, or a UI redesign.
- Plan before implementing.
- `.mcp.json` (Jira MCP config) is never committed -- see `.mcp.json.example`
  for the shape to copy from. `JIRA_API_TOKEN` is read from your shell
  environment at launch, never written into the file itself.

---
> Source: [fatirmalik1/smart-file-organizer](https://github.com/fatirmalik1/smart-file-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
