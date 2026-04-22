## finops-buddy-agent

> Prompt to consider bumping patch version and generating a new build after changes.


# Version bump and build after changes

When an **OpenSpec change implementation is complete**, or when **archiving a change** / **merging to main**:

- **Required:** Include the version-bump prompt in the **same** response as the archive/implementation summary. Do not finish the archive or implementation-complete flow without asking.
- **Prompt the user:** "Should the project patch version be bumped in `pyproject.toml` and a new build generated?"
- Do not bump the version or run the build unless the user confirms.
- If the user says yes: (1) If the change touched `frontend/` or `src/finops_buddy/webui/`, run `cd frontend && npm run build:hosted` first so the Python package includes the latest UI. (2) Update the `version` field under `[tool.poetry]` in `pyproject.toml` (e.g. `0.1.0` → `0.1.1` for a patch bump), then run `poetry build` (or `poetry lock` first if dependencies changed) to generate a new build. (3) If you run the app from an installed package in a local Poetry environment (not just `poetry run` against sources), run `poetry install` after the bump/build so `importlib.metadata.version("finops-buddy")` and similar version lookups reflect the new version in that environment. Docker image rebuilds already reinstall dependencies as part of their build steps; run Docker build after frontend and poetry build so the image gets the same `src/` with built webui.

This keeps version and builds intentional and user-controlled.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amalaguti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
