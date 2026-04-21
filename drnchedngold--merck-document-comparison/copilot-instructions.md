## merck-document-comparison

> Merck Document Comparison — playbook-first workflow and feature branches (always)


# Before any implementation

1. **Open and follow** `docs/TEAM-WORKFLOW.md` and `docs/TEAM-PLAYBOOK.md` (source of truth for process).
2. **Task kickoff:** Use the **Task kickoff context refresh** prompt from `docs/AI-PROMPTS-INDEX.md` so scope, docs, and repo layout are clear *before* writing code.
3. **Jira-driven work:** Require issue **key**, **title/summary**, and **description/AC** in chat (or confirm the user is only doing a doc/process change with no ticket).

# Git and branching (non-negotiable)

- **Do not** implement task work on `main`. **Create or checkout** a feature branch first: `{KEY}-{number}-{kebab-title}` (see playbook).
- **Before the first code edit** for a ticket: run `git status`, confirm you are **not** on `main`, or create the branch from updated `main` (`git fetch` / `git pull` as appropriate).
- **Do not** `git commit` or `git push` until the user asks to ship, except when they explicitly say to commit on the branch as part of the task.
- **Never** push task commits directly to `main`. **Ship** only when the user says e.g. “Push and open a PR” / “Ship this” — then push the **feature branch** and open a PR into `main`.

# If the user skips playbook context

**Stop** and summarize what the playbook requires for this task (branch name, kickoff prompt, scope), then proceed only after alignment—or implement only what they explicitly scoped.

# Golden / baseline changes

If emit or alignment behavior changes **ins/del** counts, plan **`python scripts/refresh_golden_corpus_baseline.py`** (or a documented partial update) **on the feature branch** before merge; do not surprise CI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DrnchedNGold) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
