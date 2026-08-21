## efficient-esmc

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Code review workflow (required)

This repo uses **roborev** for AI code review. It is configured to review with
the **`codex`** agent on model **`gpt-5.5`** at **`maximum`** reasoning
(Codex effort `xhigh`). The daemon and a post-commit hook are already installed.

When you finish a unit of work, follow this loop:

1. **Break the changes into logical commits.** Do not bundle unrelated changes.
   Each commit should be one coherent, self-contained change (e.g. "add data
   loader", "fix masking off-by-one", "add Gemma wrapper") with a clear message.
   Stage selectively (`git add -p` / per-file) so each commit stands on its own
   and builds/reads cleanly in isolation.

2. **Run a roborev review on each commit** via the `/roborev-review` skill,
   passing the commit SHA. Review commits individually rather than reviewing a
   range or branch in one shot, so findings map to a specific commit:

   ```
   /roborev-review <commit-sha>
   ```

   Equivalent direct command if invoking by hand:

   ```
   roborev review <commit-sha> --wait
   ```

   The configured agent/model/reasoning (codex / gpt-5.5 / maximum=xhigh) are
   applied automatically — do not override them unless the user asks.

3. **Present the verdict and findings** for each commit (grouped by severity,
   with file:line). If a commit has findings, offer to address them with
   `/roborev-fix <job_id>`. Resolve findings before moving on when practical.

Note: the post-commit hook also enqueues an automatic review on every commit, so
reviews will run even if you forget — but you are still responsible for
explicitly running `/roborev-review` per commit and surfacing the results.

### Useful roborev skills / commands

- `/roborev-review [commit]` — review a single commit (use this per commit)
- `/roborev-review-branch` — review all commits on the branch vs. main
- `/roborev-fix <job_id>` — apply fixes for a review's findings
- `/roborev-refine` — iterative review→fix→re-review loop until passing
- `roborev status` — daemon/queue health; `roborev list` — recent jobs

---
> Source: [cyrusmaher/efficient-esmc](https://github.com/cyrusmaher/efficient-esmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
