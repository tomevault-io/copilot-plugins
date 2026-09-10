## causaltune

> This repo enforces the **Developer Certificate of Origin (DCO)** in CI. Every

# CLAUDE.md — causaltune

## Committing: DCO sign-off is REQUIRED

This repo enforces the **Developer Certificate of Origin (DCO)** in CI. Every
commit MUST carry a `Signed-off-by:` trailer matching the commit author, or the
DCO check fails and the PR is blocked.

ALWAYS commit with `-s` (sign-off):

```bash
git commit -s -m "your message"
```

This appends `Signed-off-by: Egor Kraev <egor.kraev@gmail.com>` (from
`git config user.name` / `user.email`). When amending or rebasing, keep the
trailer — use `git rebase --signoff` or `git commit -s --amend` as needed.

Never create a commit here without the sign-off trailer.

## Do NOT commit simulation run results

`notebooks/RunExperiments/EXPERIMENT_RESULTS_*/`, `RunDatasets/`, and the
`runners/plots/` + `runners/tex/` output dirs are large generated artifacts
(tens of GB) and are gitignored. Only source code (`.py`, tests, docs,
`runners/*.py` scripts) gets committed.

---
> Source: [py-why/causaltune](https://github.com/py-why/causaltune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
