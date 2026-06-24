## eval-engineer

> This file is repo-local guidance for Codex, Claude Code, and other coding

# Agent Instructions

This file is repo-local guidance for Codex, Claude Code, and other coding
agents working on Eval Engineer. Keep it short and update it whenever the repo
workflow, source-of-truth files, or recurring operating rules change.

## Project Intent

Eval Engineer is a general Galileo evidence workflow for improving AI agents,
RAG apps, and future AI systems. Do not shape the product around the first
support-agent fixture. The support-agent cases are validation fixtures, not the
skill's scope.

The near-term product goal is to reduce time-to-RCA for agent builders and
owners while increasing Galileo discoverability for non-developer personas.
Prefer RCA workflows that query log streams, inspect traces/sessions/spans,
identify failure patterns, compare behavior over time, and return grounded
answers with links or stable IDs back to Galileo data.

The north-star loop is:

1. run the AI app
2. log traces and metrics to Galileo
3. fetch compact evidence
4. diagnose failure
5. propose a bounded fix
6. verify with local and Galileo evidence
7. keep only changes that improve measured behavior

## Read First

- `docs/plan.md`: product direction and architecture.
- `docs/tasks.md`: current task checklist and Linear issue mapping.
- `docs/progress.md`: latest work completed and next move.
- `.galileo/learnings.md`: repo-specific durable learnings.
- `blogs/`: product thinking; useful for design intent, but do not treat as
  runtime instructions.

## Skill Rules

- Canonical skill source: `skills/eval-engineer/`.
- Codex install link: `.agents/skills/eval-engineer`.
- Claude install link: `.claude/skills/eval-engineer`.
- Public installer CLI: `eval-engineer` from `pyproject.toml`. Keep it runnable
  through `uvx --from git+https://github.com/Galileo-Agent-Labs/eval-engineer.git`.
- Keep skill distribution skill-first. Future Codex and Claude plugins should
  package the canonical skill source instead of maintaining separate copies.
- Keep `SKILL.md` general across agents, RAG, workflows, metrics, and providers.
- Keep RCA outputs grounded in trace, span, session, metric, dataset, and
  experiment evidence.
- For reference fixtures, give each case a risk profile, quality dimensions,
  and case-specific Galileo metric profile. Do not rely on one global metric
  list to prove safety, quality, performance, and cost across all cases.
- Use `skills/eval-engineer/references/metric-profile-checklist.md`
  and `skills/eval-engineer/assets/metric-profile-template.md` before
  optimizing cost or adding broad fixture coverage.
- When logging test-suite cases to Galileo, include the full expected-output
  contract in `dataset_output`: expected decision, required/forbidden citations,
  required tools, answer constraints, abstention/permission requirements, risk
  profile, quality dimensions, and intended metrics.
- Use `skills/eval-engineer/references/rca-recipe.md` for generalized
  diagnose-fix-verify work and update it when a reusable Galileo RCA pattern is
  discovered.
- When improving command skills, keep frontmatter descriptions trigger-focused,
  keep `eval-engineer` as a narrow router, load references conditionally, and
  promote recurring Galileo mistakes into focused skill gotchas and validation
  loops.
- Keep detailed Galileo mechanics in `skills/eval-engineer/references/`.
- Keep deterministic helpers in `skills/eval-engineer/scripts/`.
- Do not hardcode `TC-1`, the Nexus support agent, Brazil, one model, or one
  metric into the general skill.

### Skill Package Hygiene

- Treat skill frontmatter as routing surface, not documentation. Descriptions
  should name the user job and evidence objects that should trigger the skill:
  Galileo URLs, traces, sessions, log streams, metrics, datasets, experiments,
  tokenomics, RCA, audit, setup, and measurement. Compact wording is good only
  when these trigger nouns survive.
- Keep the router and command skills non-overlapping. `eval-engineer` routes and
  reports workspace status; `eval-fetch` retrieves evidence; `eval-diagnose`
  performs RCA; `eval-measure` defines metric contracts; `eval-dataset` turns
  failures into cases; `eval-cost` handles tokenomics; `eval-audit` reviews risk
  and launch readiness; and so on.
- Budget `SKILL.md` for the workflow an agent must follow after the skill
  triggers. Move Galileo API mechanics, schemas, long examples, and edge-case
  notes to `references/`; move repeatable parsing, summarizing, and comparison
  work to `scripts/`.
- Before merging, renaming, deleting, or splitting skills, verify the kept copy
  is present in the canonical source, the Codex and Claude install links, and
  the installer bundle. Check current usage evidence such as skill mentions,
  `SKILL.md` reads, local install roots, and test coverage before deciding a
  skill is unused.
- Do not delete ignored or untracked skill directories unless the replacement
  path is named or the user confirms they are disposable.

## Working Set

- `.galileo/config.yml`: agent type, metrics, editable files, verification
  commands.
- `.galileo/current/`: current evidence and working artifacts.
- `.galileo/sessions/`: historical evidence.
- `.galileo/eval-dataset/`: candidate, accepted, and rejected eval cases.
- `.galileo/learnings.md`: durable patterns discovered while working.

Read `.galileo/current/` by default. Do not scan historical raw sessions unless
the user asks for history or comparison.

## Verification

After skill changes, run:

```bash
PYTHONPYCACHEPREFIX=/private/tmp/eval-engineer-pycache python3 -m unittest tests.skills.test_eval_engineer_skill
for skill in skills/eval-*; do python3 /Users/pratik/.codex/skills/.system/skill-creator/scripts/quick_validate.py "$skill"; done
```

For the generic packet summarizer:

```bash
python3 skills/eval-engineer/scripts/summarize_debug_packet.py tests/skills/fixtures/generic-rag-debug-packet.json
```

For tokenomics before/after packet comparison:

```bash
python3 skills/eval-engineer/scripts/compare_tokenomics_packets.py .galileo/current/debug-packet.json .galileo/current/verification-debug-packet.json --quality-metrics average_tool_selection_quality,tool_error_rate,average_completeness_gpt,average_groundedness --lower-is-better-quality-metrics tool_error_rate
```

For tokenomics work, compare cost, latency, and token movement against Galileo
quality metrics. Do not keep a cost reduction on local scoring alone, and do
not treat lower traffic volume as a per-trace efficiency improvement. Check
segment metrics when available before accepting an aggregate cost win. For RAG
retrieval pruning, include at least one hard multi-source or multi-hop case so
top-1 retrieval shortcuts cannot pass on easy single-document questions alone.
For agentic workflows, also compare agent steps, planner spans, rerank passes,
self-check spans, and tool calls so an adaptive optimization shows which part
of the loop got cheaper.

Use live model or Galileo calls only when the task explicitly requires runtime
verification. Do not print secret values.

After installer changes, test the Python CLI and a real `uvx` install into a
throwaway project:

```bash
PYTHONPYCACHEPREFIX=/private/tmp/eval-engineer-pycache python3 -m unittest tests.installer.test_install_cli
mkdir -p /tmp/eval-engineer-install-test
uvx --from /Users/pratik/Documents/github/eval-engineer eval-engineer install --target both --scope project --project-dir /tmp/eval-engineer-install-test
uvx --from /Users/pratik/Documents/github/eval-engineer eval-engineer check --target both --scope project --project-dir /tmp/eval-engineer-install-test
```

## Linear Hygiene

Keep Linear up to date manually after response completion when task status
changes. There are no repo hooks for this.

- Create Linear issues for new meaningful work.
- Move completed Linear issues to Done.
- Move obsolete issues to Canceled or Backlog with an explanatory comment.
- Keep `docs/tasks.md` issue IDs aligned with Linear.
- Mention the relevant Linear IDs in `docs/progress.md`.

Current active planning issues:

- `GAL-87`: answer-quality check for policy explanation correctness.
- `GAL-93`: Eval Engineer launch blog.
- `GAL-94`: Eval Engineer launch graphics.
- `GAL-108`: evaluate separate Codex and Claude plugin packaging.

## Secrets And Generated Files

- Never print `.env` values. It is okay to list variable names.
- Keep generated runs, raw traces, temporary logs, and `.omc/` out of git.
- Preserve user changes. Do not revert unrelated edits.

## Maintenance Rule

If you discover a new durable workflow rule, repeated mistake, packaging
decision, or verification command, update this `AGENTS.md` in the same change.

---
> Source: [Galileo-Agent-Labs/eval-engineer](https://github.com/Galileo-Agent-Labs/eval-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
