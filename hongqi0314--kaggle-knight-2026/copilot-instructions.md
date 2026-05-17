## kaggle-knight-2026

> - **Run Script:** `uv run python <script>`

# CLAUDE.md

## Commands
- **Run Script:** `uv run python <script>`
- **Start Notebook:** `uv run marimo edit notebooks/<name>.py --watch`
- **Install Package:** `uv add <package>`
- **Update Lockfile:** `uv sync`
- **Agent Commit:** `/commit "<message>"`
- **Publish Repo:** `gh repo create --private --source=. --remote=origin`
- **Push:** `git push origin main`

## Guidelines
- **Git Style:** Use Conventional Commits.
  - `feat:` New features, models, or pipelines.
  - `fix:` Bug fixes.
  - `data:` Data processing, feature engineering, or dataset changes.
  - `docs:` Documentation or log updates.
  - `chore:` Tooling, configuration, or maintenance.
  - **Do not default to `chore:`.** Pick the most specific type (`feat|fix|data|docs|chore`) based on the actual change.
  - **Commit messages must not be auto-prefixed with `chore:`** unless the change is truly maintenance/tooling.
  - **No co-author trailers by default.** Do not add `Co-authored-by:` lines unless explicitly requested.
  - **Single-author policy (default):** author/committer should be the repository owner only.
  - **Before push, verify last commit message and author:**
    - `git log -1 --pretty=format:'%h %s%nAuthor: %an <%ae>%nCommitter: %cn <%ce>'`
- **Tooling:**
  - **Execution:** Always prioritize `uv run` for consistent environments.
  - **Data Processing:** Use `import polars as pl` for memory efficiency and speed. Prefer Polars' lazy API (`.lazy()`) for heavy operations.
- **Project Structure:**
  - `data/`: Ignored by Git. Contains `raw/` and `processed/`.
  - `notebooks/`: Interactive development. Use `.py` format via Marimo or Jupytext for better diffs.
  - `src/`: Reusable modules (e.g., `src/preprocessing`, `src/models`).
- **Experiment Tracking:**
  - Summarize all key experiment results (Model, CV Score, LB Score, Params) in `EXPERIMENTS.md` before committing.

---
> Source: [hongqi0314/kaggle_knight_2026](https://github.com/hongqi0314/kaggle_knight_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
