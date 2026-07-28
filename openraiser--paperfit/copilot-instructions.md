## paperfit

> PaperFit is a cross-host academic-paper layout agent system. Use it when the user wants to analyze or improve LaTeX paper layout, repair visual defects, migrate templates, adjust page budget, or inspect tables/figures.

# PaperFit for Codex

PaperFit is a cross-host academic-paper layout agent system. Use it when the user wants to analyze or improve LaTeX paper layout, repair visual defects, migrate templates, adjust page budget, or inspect tables/figures.

## Product Model

- Treat PaperFit as a **natural-language agent system**, not a CLI-first tool.
- The user should describe the goal directly, for example: `Analyze this paper's layout`, `Migrate this paper to CVPR`, `Reduce the paper to 8 pages with minimal semantic change`.
- You should automatically route the request into the appropriate PaperFit workflow: layout analysis, full VTO repair, visual-only inspection, template migration, length adjustment, table repair, or status review.

## What To Use

- Primary Codex entry: the custom agent `paperfit` installed at `~/.codex/agents/paperfit.toml`.
- Use skills under `~/.codex/skills/paperfit/` as the PaperFit capability bundle.
- Treat `skills/paperfit/agents/` as role references and workflow documentation.
- Use `paperfit` CLI, runtime, and bundled scripts as the **internal execution layer** for rendering, state transitions, parsing, and repair execution.

## Preferred Workflow

1. Work from the paper project root.
2. Infer the task from the user's natural-language request before asking for explicit PaperFit command syntax.
3. Use PaperFit internal execution automatically when needed; do not require the user to type `paperfit run` or `paperfit runtime`.
4. Run `paperfit doctor --target codex` only when environment health is unclear.
5. Prefer `paperfit runtime --state data/state.json run-round main.tex --template <TEMPLATE> --target-pages <N>` as the default closed-loop executor.
6. Fall back to `paperfit render`, `paperfit run scripts/parse_log.py`, and other low-level scripts only for targeted debugging or staged recovery.

## Interaction Rules

- During long PaperFit runs, proactively report progress instead of staying silent.
- In recoverable failures, prefer self-recovery first. Only ask the user for action when the environment is broken, permissions are missing, or the target is genuinely ambiguous.
- Frame user-facing updates around results, visual defects, repair strategy, and residual risks, not around internal commands.

## Non-Negotiable Constraints

- Do not claim layout success without checking rendered page images.
- Use visual results as the primary acceptance signal; source and logs are supporting evidence.
- Do not use `\resizebox` or `\scalebox` as brute-force table compression.
- Prefer full-width or full-column table reconstruction strategies such as `table*`, `tabularx`, or other width-aware layout fixes.
- Do not change the paper's template font packages or the visual font appearance of body text and tables during repair.
- Preserve academic meaning; semantic edits must stay minimal, controlled, and auditable.
- Do not silently delete figures, tables, captions, labels, or other key paper objects to improve layout.

## Host Notes

- Codex uses `AGENTS.md` for persistent project instructions.
- Ask Codex to use or spawn the `paperfit` agent by name for PaperFit work.
- `/agent` in Codex switches to an existing agent thread; it is not a Claude-style custom slash-command menu.
- Codex custom prompts are deprecated upstream, so they are not the primary PaperFit entry.
- PaperFit does not own the model provider. It inherits the user's Codex runtime and `~/.codex/config.toml`.
- Third-party or self-hosted OpenAI-compatible gateways are valid as long as the user's Codex CLI already works with them.

---
> Source: [OpenRaiser/PaperFit](https://github.com/OpenRaiser/PaperFit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
