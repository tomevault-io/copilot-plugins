## no-one-did-it

> > **Single source of truth.** This file is loaded by Claude (`CLAUDE.md` → `@AGENTS.md`), Codex (`AGENTS.md` directly), and Gemini (`GEMINI.md` → `@AGENTS.md`). All three tools share this context. Edit here; never edit `CLAUDE.md` or `GEMINI.md` directly.

# Responsibility Laundering Book — Project Instructions

> **Single source of truth.** This file is loaded by Claude (`CLAUDE.md` → `@AGENTS.md`), Codex (`AGENTS.md` directly), and Gemini (`GEMINI.md` → `@AGENTS.md`). All three tools share this context. Edit here; never edit `CLAUDE.md` or `GEMINI.md` directly.

Project: **No One Did It: Responsibility Laundering, from the Scapegoat to the Algorithm**.

Use this workspace to research, structure, draft, audit, and market a serious trade nonfiction book about responsibility laundering.

## Global Five Over-Rules

<!-- GENERATED:five-over-rules:start -->
1. **Evidence before elegance.** Never improve the story by weakening the evidence.
2. **Responsibility follows control, benefit, knowledge, and preventability.** Do not stop at the most visible actor.
3. **Keep the taxonomy intact.** Distinguish pure scapegoat, partial scapegoat, system/object alibi, and cost-bearing goat.
4. **Steelman before judgment.** Every major claim must face its strongest counterargument before it is asserted.
5. **Handoff cleanly.** Every output must state assumptions, evidence grade, open questions, and next owner.
<!-- GENERATED:five-over-rules:end -->

## Core book thesis

Civilization did not stop sacrificing substitutes. It changed the altar. Responsibility laundering is the recurring method by which power keeps control or benefit while moving blame, liability, moral cost, or public suspicion onto a weaker bearer, procedural shell, proxy, machine, market, committee, legal entity, or record.

## Core diagnostic

For every case ask:

1. Who or what was publicly blamed?
2. Who had control?
3. Who benefited?
4. Who knew or should have known?
5. Who could have prevented recurrence?
6. Who controlled the record?
7. Who bore the cost?
8. What would responsibility look like if it followed control instead of visibility?

## Case taxonomy

- **Pure scapegoat:** mostly innocent actor is blamed to close the case.
- **Partial scapegoat:** actor is involved or guilty at one level, but blame stops too low and protects a wider chain.
- **System/object alibi:** machine, model, benchmark, corporation, committee, market, legal form, or procedure becomes the blame container.
- **Cost-bearing goat:** victim or worker is not accused as guilty but absorbs harm while responsibility is avoided.

## Style rules

- Use narrative heat for scenes and analytical coldness for conclusions.
- Do not invent scenes, dialogue, motives, or thoughts.
- Do not make all cases morally identical.
- Do not create partisan symmetry or false equivalence.
- Do not let novelty outrank verifiability.
- Prefer official records, court files, inquiries, watchdog reports, primary documents, and high-quality investigative sources.
- Flag every live/contested case as such.

## Model policy

**All 13 crew agents run on `opus`.** Principal Author directive: logic-laden tasks require the larger model. The crew does no purely-mechanical work — every role involves multi-step reasoning, judgment under constraints, or reconciliation of contradictory inputs. Sonnet is reserved for ad-hoc utility tasks outside the crew (e.g., one-shot prose tweaks where logic is not load-bearing).

Verified at every test run by the per-agent `model:` frontmatter check; any drift to sonnet on a crew agent should fail review.

## Default artifact standard

Every substantial output should include:

```text
Owner:
Purpose:
Evidence grade:
Assumptions:
Open questions:
Handoff:
```

## Repository layout

All inputs and outputs referenced by the crew now live inside this project
directory (`books/responsibility_laundering/`). The previous artifact-bundle
ZIPs were extracted and relocated under `dev-docs/`; the active crew
workspace was lifted from `dev-docs/03_CLAUDE_AI_CREW_WORKSPACE/extracted/responsibility-laundering-book-ai-crew/`
to the project root.

### Active workspace (project root)

- `AGENTS.md` — this file; the single source of truth for all three tools.
- `CLAUDE.md` / `GEMINI.md` — one-line `@AGENTS.md` imports; do not edit directly.
- `README.md` — crew workspace quick-start.
- `.claude/agents/` — the 13-agent AI crew (Jerry, Bonnie, Wayne, Delon, Shirley, Selina, Warren, Loki, Stephen, Laura, Nancy, Alan, Blair). See `.claude/docs/crew-portfolio.md` for the full roster.
- `.claude/skills/` — 14 project-owned skills (`case-file-method`, `counter-case-method`, `chapter-blueprint`, `responsibility-chain-mapping`, `source-ledger-discipline`, `citation-hygiene`, `counterargument-red-team`, `publication-proposal`, `evidence-grading`, `taxonomy-classification`, `defamation-wording`, `scene-construction`, `primary-source-playbooks`, `vocabulary`) plus the `.claude/skills/cc-suite/` symlink (→ `~/.claude/plugins/cache/xiaolai/cc-suite/<ver>/skills/cc-suite`) installed by `/cc-suite:bridge-skills` so Codex sees plugin-provided sub-skills via `.agents/skills`. The plugin symlink is bridge infrastructure, not a project skill, and is excluded from NLPM scans (no top-level `SKILL.md`).
- `.claude/commands/` — slash commands (`/crew-briefing`, `/case-file`, `/chapter-brief`, `/source-audit`, `/red-team`, `/proposal-pack`, `/figure-spec`, `/photo-clear`).
- `.claude/rules/` — five over-rules, evidence grades, taxonomy, role map, style guide.
- `.claude/hooks/` — session-context, destructive-bash guard, agent-frontmatter check, subagent-finish log.
- `.claude/docs/` — agent role map, templates (`case-card-template.md`, `chapter-template.md`, `source-ledger-template.md`), workflow, hooks guide, references.
- `.claude/settings.json` — project hook configuration.

### Book outputs (project root)

- `book/evidence/case-files/` — case cards and chronologies.
- `book/evidence/source-ledger/` — source ledgers and claim logs.
- `book/chapters/` — chapter briefs and drafts.
- `process/review-memos/` — fact-check, red-team, legal, and expert memos.
- `book/proposals/` — proposal, query, comp-title, and sample-chapter materials.
- `book/evidence/diagrams/` — responsibility-chain maps, charts, and tables (visual frameworks). Spec owned by Bonnie; data graded by Stephen. See rule `03` "Visual material ownership."
- `book/evidence/photos/` — photographs and composites (created when first needed). Rights + caption clearance gated by Nancy; provenance verified by Stephen. See rule `03`.

### Relocated reference artifacts (`dev-docs/`)

Treat as read-only input material; do not regenerate. When the source-ledger
or case matrix already covers a claim, prefer it over fresh searches.

- `dev-docs/00_START_HERE/` — `START_HERE.md`, `OFFLINE_STARTUP_WORKFLOW.md`, `ARTIFACT_MANIFEST.md`, `DIRECTORY_TREE.md`, `artifact_inventory.csv`, `checksums_sha256.txt`. (Path prefixes inside these files are pre-relocation: anything they refer to as `01_BOOK_PREPARATION_DECKS/…` now lives at `dev-docs/01_BOOK_PREPARATION_DECKS/…`, and similarly for `02_`, `03_`, `04_`.)
- `dev-docs/01_BOOK_PREPARATION_DECKS/extracted/book_prep_package/` — `01_Strategy_and_Audit_Deck.pptx`, `02_Historical_Case_File_Deck.pptx`, `03_Narrative_and_Chapter_Deck.pptx`, `Audit_Memo.md`, `Offline_Workplan.md`, `Source_Anchors.md`.
- `dev-docs/02_RESEARCH_PACKAGES/extracted/responsibility_laundering_research_packages/` — `master_case_matrix.csv|json`, `source_ledger_master.csv`, plus per-domain packets `00_Methodology/`, `01_War_Ukraine_Iraq/`, `02_Trump_Administrations/`, `03_AI_Competition/`, `04_Book_Integration/`.
- `dev-docs/03_CLAUDE_AI_CREW_WORKSPACE/` — original ZIP and the extracted copy of the *seed* crew workspace (with the fictional-name 21-agent crew). The live crew at the project root replaces those names with the working identifiers used here and consolidates to 13 agents.
- `dev-docs/04_ARCHIVE_SUPERSEDED_OR_REFERENCE/extracted/claude_book_ai_crew_workspace/` — earlier workspace variant with different crew names and rule numbering. Reference only; do not import its agents or rules into the active workspace without explicit migration.

### Path-reference rule

When citing any of the above in case files, chapter briefs, memos, or
handoffs, use the project-root-relative path (e.g.
`dev-docs/02_RESEARCH_PACKAGES/extracted/responsibility_laundering_research_packages/master_case_matrix.csv`),
not the pre-relocation path from inside the bundle's own README files.

## Prerequisites

- Python 3.10 or newer (for `pytest`, the scripts in `scripts/`, and the project hooks).
- `PyYAML` (`python3 -m pip install PyYAML`) — used by the workshop validators that this project inherits.
- Claude Code with the `eou-foundry@xiaolai` plugin enabled at the workspace level (inherited from `book-workshop/.claude/settings.json`) and the `nlpm@xiaolai` plugin available for the corpus quality gates.
- Optional: Codex CLI and Gemini CLI. Their bridge slots (`.codex/`, `.gemini/`, `.agents/`) are populated by the `cc-suite` plugin when enabled; they stay empty otherwise.

## Build and verify

No build step. The project is verified by a deterministic test and validator suite, run from this directory:

```bash
# Run all 66 project tests (frontmatter, hooks, graph, NLPM specs, rhythm gate, sync checks).
python3 -m pytest tests/ -q

# Verify the Global Five Over-Rules block has not drifted across the 28 targets that embed it.
python3 scripts/sync_five_over_rules.py --check

# Verify the bounded delegation DAG (depth ≤ 2 from jerry-crew-chief) and tool-grant policy.
python3 scripts/check_agent_graph.py
python3 scripts/check_tool_grants.py
```

The workshop-level validator (`python3 scripts/validate_workshop.py` in `book-workshop/`) wraps these and the Foundry plugin's own validators.

## Shared Memory

**Always record new instructions, rules, and memory in `AGENTS.md` only.**

Never modify `CLAUDE.md` or `GEMINI.md` directly — they only import `AGENTS.md`.
This keeps Claude Code, Codex CLI, and Gemini CLI on the same context.

## Project Structure

- `.claude/` — Claude Code skills, agents, rules, hooks, commands
- `.agents/skills/` — symlink to `.claude/skills/` (Codex skill scan path)
- `.codex/prompts/` — slot for Codex slash-command prompts (empty by default; populate only when Codex slash-command bridging is required)
- `.codex/hooks.json` / `.codex/config.toml` — Codex hooks/config (optional)
- `.gemini/skills/`, `.gemini/commands/` — Gemini skills and TOML commands
- `.mcp.json` — MCP server registrations (shared by all three tools)

---
> Source: [xiaolai/no-one-did-it](https://github.com/xiaolai/no-one-did-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
