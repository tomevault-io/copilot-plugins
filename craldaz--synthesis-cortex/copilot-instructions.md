## synthesis-cortex

> This file is the authoritative guide for all AI-assisted development in this project.

# Synthesis Cortex — Gemini CLI Workspace

This file is the authoritative guide for all AI-assisted development in this project.
Read it in full before writing any code or modifying any file.

---

## 1. Project Overview

The Synthesis Cortex is a personal knowledge management (PKM) automation system built on top of an Obsidian vault, Things 3, and external APIs (Infranodus, Anthropic/Gemini, ReadCube). It transforms the vault from a static note store into an active research engine with three core capabilities:

1. **The Populator** (`scripts/populator.py`): Creates structured foundation notes from Things 3 tasks tagged `#obsidian-note`.
2. **The Interrogator** (`scripts/vault_gap_scanner.py`): Finds structural holes between knowledge domains using Personalized PageRank + Infranodus, then generates bridging research questions.
3. **The Retriever** (`scripts/retriever/` — *in development*): A multi-layer hybrid search system that answers queries against the ReadCube library using symbolic (label-based), dense (embedding-based), and graph-aware retrieval.

The strategic north star is documented in `docs/PKM Strategy Roadmap.md`. Read it to understand the full picture before working on any component.

---

## 2. Repository Layout

```
.
├── docs/                            # Strategy, planning, and reference docs
│   ├── PKM Strategy Roadmap.md      # ← Master strategy document. Read first.
│   ├── Automating Knowledge Synthesis Workflows.md
│   ├── vault_gap_scanner_future_directions.md
│   ├── FEEDBACK_PROTOCOL.md             # How gap reports are reviewed and fed back to DSPy
│   │── PROGRESS.md                      # Running log of completed milestones
│   └── ...
├── scripts/
│   ├── build_manifest.py            # Builds domain manifest via Personalized PageRank
│   ├── vault_gap_scanner.py         # Core gap detection + Infranodus + LLM pipeline
│   ├── populator.py                 # Claude-backed note creation from Things 3
│   ├── populator_gemini.py          # Gemini-backed variant of populator
│   └── dspy_engine/
│       ├── dspy_gap_compiler.py     # DSPy signatures, modules, and optimizer
│       ├── parse_gap_reports.py     # Extracts structured data from gap-report-*.md files
│       ├── trainset.json            # Training data for DSPy optimizer
│       └── models/
│           └── optimized_gap_generator.json  # Compiled DSPy program state
├── GEMINI.md                        # ← You are here
├── README.md
└── requirements.txt
```

**Planned additions (not yet scaffolded):**
- `scripts/sync_readcube_stubs.py` — ReadCube bib → Obsidian stub notes
- `scripts/synthesis_debt_detector.py` — Finds read-but-unsynethsized papers
- `scripts/retriever/` — Multi-layer hybrid reference retrieval system

---

## 3. Environment Setup

Always verify the environment before executing any script.

### Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Required Environment Variables

All scripts source these from the environment. Never hardcode keys.

| Variable | Required By | Purpose |
|---|---|---|
| `VAULT_PATH` | All scripts | Absolute path to Obsidian vault |
| `ANTHROPIC_API_KEY` | `populator.py`, retriever | Claude API access |
| `GEMINI_API_KEY` | `populator_gemini.py`, DSPy modules, `vault_gap_scanner.py` | Gemini API access |
| `INFRANODUS_API_KEY` | `vault_gap_scanner.py` | Graph analysis API |

Verify before running:
```bash
echo $VAULT_PATH && echo $ANTHROPIC_API_KEY && echo $GEMINI_API_KEY && echo $INFRANODUS_API_KEY
```

---

## 4. Core Architecture Principles

These rules are non-negotiable. They protect the integrity of the vault and the knowledge graph.

### 4.1 Never Break the Graph

`build_manifest.py` constructs a NetworkX directed graph from every wikilink in the vault. All domain discovery depends on this graph. Rules:
- Do not introduce folder-based routing logic into manifest generation. Folder location is an unreliable proxy for semantics; the link graph is the ground truth.
- Cross-folder links must be preserved. Do not filter edges by source or target directory.
- `EXCLUDE_DIRS` exists to prevent meta-documents (gap reports, copilot logs) from acting as artificial bridges. Respect it and extend it carefully.

### 4.2 Protect the Vault

The vault is a source of truth accumulated over years. Any script that writes to `VAULT_PATH` must:
- Use atomic writes (write to a temp file, then rename)
- Never delete files without an explicit `--dry-run` → user approval → `--execute` workflow
- Never perform bulk modifications without showing a diff-style preview first
- Stub notes (from ReadCube sync) must be written to a dedicated `_stubs/` subdirectory, not mixed with concept notes

### 4.3 Determinism and Robustness

- All functions must have Python type hints (PEP 484)
- All API calls must have retry logic with exponential backoff and a max retry limit
- All Markdown parsing must handle malformed frontmatter gracefully (try/except around yaml.safe_load)
- No script should silently swallow exceptions. Log errors explicitly, then decide whether to continue or abort
- Prefer returning structured types (dataclasses or TypedDict) over raw dicts for any data that crosses function boundaries

### 4.4 Human Synthesis Is Irreplaceable

The concept notes in the vault represent human-authored synthesis. LLM-generated content must never be mixed silently into concept notes. The permitted pattern is:
- LLM output goes into a clearly delimited section (e.g., `## Recent Literature`) or a separate stub file
- The human decides whether to promote it into the main body
- This applies to the Populator, the stub enrichment workflow (Idea 4 in the roadmap), and any future auto-annotation

---

## 5. Component Reference

### 5.1 `build_manifest.py` — Domain Manifest Builder

Constructs a YAML manifest mapping semantic domains to ranked note lists via Personalized PageRank on the vault link graph.

**Key concepts:**
- **Anchors:** 3–5 hand-chosen notes per domain that seed the PageRank walk
- **Goldilocks pairs:** Domain pairs with Jaccard overlap 0.02–0.40 in their top-25 notes; these are the structurally interesting pairs for gap detection
- **`EXCLUDE_DIRS`:** Path prefixes excluded from graph construction to prevent meta-document contamination

**Do not modify** the PageRank seeding logic or Jaccard thresholds without benchmarking the effect on gap question quality. Changes here affect every downstream component.

**Planned upgrade (Roadmap Item 2):** Add abstract-weighted edges for `@`-stub notes. Stubs receive edge weight `0.15` vs. `1.0` for concept-to-concept links. Stubs are traversable but receive a `score_discount` and are not written to domain manifests.

### 5.2 `vault_gap_scanner.py` — Structural Gap Detector

The production engine. Samples notes from Goldilocks domain pairs, sends them to Infranodus, receives a structural hole analysis, generates a bridging question and hypothesis via LLM, and creates a Things 3 task.

**Rules:**
- Do not modify core logic or output format without presenting a written plan first
- Output format (gap report Markdown) must remain parseable by `parse_gap_reports.py`
- The DSPy module (once integrated) replaces only the raw LLM call, not the Infranodus pipeline

**Planned upgrade (Roadmap Item 5):** After gap question generation, automatically call the retriever in Serendipity Mode and append top-5 candidate papers to the Things 3 task.

### 5.3 `dspy_engine/` — Critique-Driven Optimizer

Implements the feedback loop that improves gap question quality over time.

**Pipeline:**
1. `parse_gap_reports.py` — reads `gap-report-*.md` files, extracts structured (domain_a, domain_b, question, hypothesis, fidelity, utility, critique) tuples into `trainset.json`
2. `dspy_gap_compiler.py` — defines `GenerateGapInsight`, `CritiqueAgainstCodysRules`, and `ReviseGapInsight` DSPy signatures; builds the `SelfCorrectingGapGenerator` module; runs MIPROv2/COPRO optimizer against the critique-aware metric; saves to `models/optimized_gap_generator.json`

**The critique-aware metric:** If a past critique exists for a domain pair, use an LLM-as-judge to verify the new prediction avoided the failure mode described in the critique. This prevents mode collapse by learning *boundaries*, not just *centers*, of acceptable output.

**Do not filter out low-scoring training examples.** Keep all examples with their critiques. Training only on 5/5 examples causes mode collapse toward conservative, predictable questions.

### 5.4 `populator.py` / `populator_gemini.py` — Note Populator

Fetches Things 3 tasks tagged `#obsidian-note`, calls the LLM to generate a structured Markdown foundation note, and writes it to the appropriate vault directory.

Generated notes must include: correct YAML frontmatter, wikilinks to related concepts, LaTeX for mathematical notation, and code blocks where relevant.

---

## 6. Planned Components (Not Yet Implemented)

Consult `docs/PKM Strategy Roadmap.md` for full specifications. Summarized here for orientation.

### 6.1 `sync_readcube_stubs.py` (Roadmap Item 1 — High Priority)

Parses the ReadCube bib export and creates lightweight `@`-prefixed stub notes in `VAULT_PATH/_stubs/`. Each stub contains: title, abstract, authors, year, ReadCube tags, and an empty `## Linked Concepts` section. After creation, runs GNN link prediction to populate `## Linked Concepts` with wikilinks to concept notes (confidence threshold ≥ 0.6).

This is the unlock for all other planned components.

### 6.2 `synthesis_debt_detector.py` (Roadmap Item 3 — High Priority)

Identifies papers that are GNN-linked to concept notes but whose abstract content is not reflected in those notes. Uses SPECTER2 embeddings. Outputs ranked (concept_note, stub, gap_score) tuples and creates targeted Things 3 tasks.

### 6.3 `retriever/` (Roadmap Item 4 — High Priority)

Multi-layer hybrid retrieval system with three retrieval modes:

| Mode | Use Case | Dominant Layer |
|---|---|---|
| Precision | Specific method/target query | Symbolic (label-based, IDF-weighted) |
| Neighborhood | "What have I read about X?" | Graph-aware (vault topology traversal) |
| Serendipity | Gap-filling, unknown unknowns | Dense (SPECTER2) + inverted graph boost |

Architecture: Layer 1 (symbolic) + Layer 2 (SPECTER2 FAISS) + Layer 3 (graph proximity boost) fused via weighted Reciprocal Rank Fusion. DSPy query classifier auto-selects layer weights.

---

## 7. The Feedback Protocol

Gap reports are the primary training signal for the DSPy optimizer. The format in every `gap-report-*.md` must be preserved exactly:

```markdown
**Eval:** fidelity=[1-5]/5  utility=[1-5]/5  — [AI self-evaluation reasoning]
**Cody's Critique:** [Nuanced explanation of why the output succeeded or failed.
This is the training signal. Explain the specific failure mode, not just the score.]
```

The `parse_gap_reports.py` script depends on this format. Do not change it without updating the parser simultaneously. See `FEEDBACK_PROTOCOL.md` for the full review workflow.

---

## 8. Workflow Conventions

### Before Starting Any Feature

1. Read `docs/PKM Strategy Roadmap.md` to confirm the feature fits the dependency graph
2. Check `PROGRESS.md` for completed work that may be adjacent
3. If modifying `vault_gap_scanner.py` or `build_manifest.py`: write a plan, present it, wait for approval

### Code Quality

- Type hints on all functions and class attributes
- Docstrings on all modules and public functions
- No new heavy dependencies without justification in the PR description
- Logging via Python's `logging` module (not `print`) at appropriate levels (DEBUG for verbose, INFO for milestones, WARNING/ERROR for failures)
- All file I/O relative to `VAULT_PATH` or the repo root — no hardcoded absolute paths

### Testing

- Scripts that modify the vault must support `--dry-run` mode that logs intended actions without executing them
- For scripts with an Infranodus or LLM call: cache API responses during development to avoid unnecessary spend
- Before merging changes to `vault_gap_scanner.py`: run a dry-run end-to-end and verify the output gap report is parseable by `parse_gap_reports.py`

### Commit Messages

```
<component>: <imperative summary>

<optional body explaining why, not what>
```

Examples:
- `build_manifest: add abstract-weighted stub edges for PageRank`
- `retriever: implement FAISS dense retrieval layer with SPECTER2`
- `dspy_engine: switch optimizer from BootstrapFewShot to MIPROv2`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/craldaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/craldaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
