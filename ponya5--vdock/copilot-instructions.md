## vdock

> Every non-trivial implementation, feature, bug fix, or design change MUST be documented in `design-log/`. Conventions are defined in `design-log/README.md`.

# Project Rules

## Design log — required for every implementation

Every non-trivial implementation, feature, bug fix, or design change MUST be documented in `design-log/`. Conventions are defined in `design-log/README.md`.

- **New work:** create `design-log/DL-<NNN>-<short-slug>.md` (next sequential number) BEFORE writing code, with a `## Implementation Results` section appended once the work is done and verified.
- **Follow-ups:** changes to an already-logged feature append a follow-up/implementation-results section to that entry instead of creating a new file.
- Sections above `## Implementation Results` are frozen once implementation begins — corrections and deviations go in the results section.
- Update the index table in `design-log/README.md` when adding a new entry.
- Trivial changes (typo fixes, one-line tweaks) are exempt; when in doubt, log it.

## Serving the frontend to the 7" panel

The Flask backend serves `frontend/dist` (see `app.py` `frontend_index`/`dist_root`). The physical touch panel loads the **built bundle**, not the Vite dev server — after frontend changes, run `cd frontend && npm run build` or the device keeps showing the stale build. The dev server on :4444 is for live verification (Playwright) only.

---
> Source: [ponya5/VDock](https://github.com/ponya5/VDock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
