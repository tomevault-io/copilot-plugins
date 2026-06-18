## acc-int-paper

> This tree is the manuscript source, not the build output.

# AGENTS

This tree is the manuscript source, not the build output.

## Source boundaries

- Edit `neurips_2026.tex`, the files under `sections/`, `tables/`, and authored
  appendix/source-note files.
- Treat generated LaTeX artifacts (`.aux`, `.bbl`, `.fls`, `.log`, `.out`,
  `.pdf`) as build products, not narrative sources.

## Paper scope

- Keep the manuscript centered on **MRH -> TanGCE as an operational constraint
  -> empirical support for MRH's operational prediction**.
- Use geometry/manifold results to motivate or explain MRH and TanGCE, but do
  not overclaim them as causal proof or as a smooth-manifold theorem.
- Treat matched ambient comparisons and prior-eraser stacking experiments as
  evidence that the tangent constraint is the useful inductive bias, not as the
  entire paper narrative.
- Be careful not to promote weak linear-transferability material into the main
  story when the tangent-vs-ambient comparison is the core claim.
- Do not claim replication or generalization from placeholder or incomplete
  tables.
- Do not state strong no-utility-cost or off-manifold-causality claims unless
  the main text directly supports them.
- Do not leave unresolved drafting notes or inconsistent dataset narratives in
  source files.

## Editing guidance

- Prefer section-local edits over monolithic changes.
- Keep notation, method, evaluation, results, and limitations aligned with the
  active GCE narrative.
- Treat tables, figures, and appendix notes as supporting material unless the
  main section text explicitly needs them.
- After each paper change, commit it promptly and push the branch to the current
  remote; keep paper changes separate from unrelated work.
- **Specifically: after every edit to any `*.tex` file within `paper/`, run
  `git push` immediately.** Do not batch paper edits with pushes.

---
> Source: [MatanAvitan/acc_int_paper](https://github.com/MatanAvitan/acc_int_paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
