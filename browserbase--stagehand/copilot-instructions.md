## stagehand

> Only a human may request changes to this file. Keep additions rare and limited to durable, repository-wide rules.

<!--
Only a human may request changes to this file. Keep additions rare and limited to durable, repository-wide rules.
-->

- For stacked PRs, target each PR at its immediate predecessor; when a parent changes, merge it into its immediate child and resolve conflicts normally.
- After a parent is squash-merged as `S`, ensure its child contains the parent’s final tip and verify that tip and `S` have identical trees. Record the child’s current commit, run `git merge --no-edit S^` and `git merge --no-edit -s ours S`, then use `git diff` to confirm that the child’s files did not change and inspect `git diff --merge-base S HEAD` as the final PR diff before pushing. Stop if either diff contains unexpected changes.

---
> Source: [browserbase/stagehand](https://github.com/browserbase/stagehand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
