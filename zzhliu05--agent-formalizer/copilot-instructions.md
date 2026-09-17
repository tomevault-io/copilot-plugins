## agent-formalizer

> This project is a lightweight Obsidian-first research vault for 构建将数学材料转化为可教学、可追踪、可由 Lean 检验内容的完整多 Agent 形式化流程.

# AGENTS.md

# 多 Agent 形式化数学教材 Research Vault Instructions

## Purpose

This project is a lightweight Obsidian-first research vault for 构建将数学材料转化为可教学、可追踪、可由 Lean 检验内容的完整多 Agent 形式化流程.

Domain: 形式化数学、多智能体协作、自动定理证明与教材工程

User background and preferred stance: 面向具备大学数学或软件工程基础、但不要求已有 Lean 经验的数学与工程协作者

Vault balance: 工程与代码优先，同时保留研究和写作区域

Explanation style: 数学直觉 → 明确规格 → Agent 输入输出 → Lean 验证

The vault is meant to accumulate a structured network of knowledge over time while remaining useful for research notes, derivations, source reading, code experiments, outputs, figures, and technical writing.

## First Rule: Decide Whether Vault Context Matters

Before answering any user prompt in this project, classify the request:

1. `standalone direct answer`: answer from current context and general knowledge.
2. `project-continuity task`: read vault context before answering.
3. `durable vault update`: read vault context, make the requested change, and update the process log.

Default to direct answers for isolated questions. Use vault context when continuity matters.

Use vault context when the request:

- references this project's prior work, files, sources, decisions, or ongoing research direction;
- asks to continue, synthesize, summarize, or revise an existing process;
- references a known topic, source, note, wiki page, code artifact, output, or figure;
- asks what has been done, decided, read, derived, or implemented;
- asks to add, reorganize, or update durable project knowledge.

For project-continuity tasks, read in this order:

1. `AGENTS.md`;
2. the newest relevant entries of `wiki/process-log.md`;
3. `wiki/index.md`;
4. linked wiki, note, raw, code, output, or figure pages as needed.

Do not read the whole process log by default. It is newest-first; start at the top and follow links outward only when needed.

## Hook-Ready Reminder

The project-local `UserPromptSubmit` hook should inject this reminder:

```text
Before answering in this project:
1. Classify the prompt: standalone direct answer, project-continuity task, or durable vault update.
2. If project continuity matters, read newest relevant entries of [[wiki/process-log]] and [[wiki/index]] before answering.
3. If durable knowledge changes, update the relevant wiki/note pages and prepend a compact entry to [[wiki/process-log]].
4. Prefer links/backlinks over long summaries.
5. If the task involves Markdown files, use the Obsidian Markdown skill for detailed formatting/linking instructions when available.
6. If the prompt belongs to an existing research process, identify its process ID from recent [[wiki/process-log]] entries or [[wiki/index]] before continuing; when starting a clearly new research process, assign the next simple process ID and mention it in the note and process log.

Do not read the whole process log by default. Read the newest relevant entries first, then follow links outward only when continuity matters.
```

Do not modify global Codex hook configuration unless the user explicitly asks for runtime hook installation.

## Vault Layout

- `raw/`: immutable source material, including papers, clipped articles, datasets, transcripts, screenshots, and source assets.
- `wiki/`: durable synthesized knowledge maintained by agents.
- `wiki/index.md`: main navigational map.
- `wiki/process-log.md`: newest-first agent memory of the research trajectory.
- `wiki/topics/`: broad research areas.
- `wiki/concepts/`: reusable concepts, definitions, and conceptual building blocks.
- `wiki/methods/`: algorithms, mathematical methods, workflows, and technical procedures.
- `wiki/papers/`: durable paper summaries and source-oriented pages.
- `notes/`: working research notes, including derivations, reading notes, speculative ideas, scratch calculations, and open questions.
- `code/`: scripts, notebooks, toy implementations, numerical experiments, and utilities.
- `outputs/`: generated artifacts such as tables, data exports, rendered summaries, and intermediate results.
- `figures/`: generated or imported figures intended for reuse in notes and wiki pages.
- `docs/`: project-level documentation and design specs.

Derivations belong in `notes/`, not in a separate derivation tree, because derivation and research interpretation are usually intertwined.

## Obsidian Conventions

Use Obsidian-style `[[Wiki Links]]` for internal links. Prefer links and backlinks over long duplicated summaries.

When a task involves creating, editing, reviewing, or formatting Markdown files for this vault, use the Obsidian Markdown skill for detailed formatting and link-handling instructions when it is available.

Use YAML frontmatter where it helps search and filtering:

```yaml
---
type: wiki | note | source | code | output | figure | process-log
status: seed | active | stable | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Use tags sparingly for broad categories only. Do not create dense tag taxonomies early.

## Human-Readable Names

Keep the approved top-level folders as they are, but make future human-facing folders and Markdown filenames easy to read in Obsidian.

For wiki pages, notes, and paper summaries, prefer descriptive names with words separated by spaces, such as `wiki/concepts/Attention Mechanism.md` or `notes/2026-05-20 Transformer Reading Notes.md`. Avoid opaque slugs, abbreviations, or version-like names when a readable title would work better.

For `raw/`, `code/`, `outputs/`, and `figures/`, preserve source filenames or use machine-safe names when reproducibility matters, but link those artifacts from readable notes or wiki pages.

## Process Log Policy

Use one global process log: `wiki/process-log.md`.

The process log is agent-first memory, not a detailed human lab notebook. Its job is to help future agents recover the research trajectory quickly: what was explored, what sources entered the vault, what topics became active, what artifacts were produced, what assumptions or conventions are currently in use, and what open processes should be resumed.

Keep the process log reverse chronological. New entries go at the top.

Use this entry shape:

```markdown
## YYYY-MM-DD - Short Status Title

Status: active | paused | resolved | superseded
Process: P001
Links: [[wiki/topic]] - [[notes/note]] - [[raw/source]]
Summary: One or two sentences about what changed.
Next: The most useful continuation step, if known.
```

Omit `Process:` only when an entry is project-system maintenance rather than research-process work. Prefer links over prose. Do not duplicate detailed derivations, paper summaries, or code explanations in the process log.

## Lightweight Process IDs

When a prompt belongs to an existing research process, identify its process ID from recent `wiki/process-log.md` entries or `wiki/index.md`.

When starting a clearly new research process, assign the next simple process ID, such as `P001`, and mention it in the note and process log. Do not create a separate process database.

## Core Workflows

### Ingest Source

Place or identify the source in `raw/`. Create or update a source-oriented page in `wiki/papers/` or a related note in `notes/`. Link the source to relevant topic, concept, and method pages. Update `wiki/index.md` if the source changes the map. Prepend a compact entry to `wiki/process-log.md`.

### Research Note

Create or update a note in `notes/`. Notes may freely mix derivation, interpretation, questions, speculation, and reading comments. Link notes back to relevant wiki pages and raw sources. Assign a process ID when the note starts a clearly new research process.

### Promote To Wiki

When a note produces stable insight, update the relevant `wiki/` page with concise synthesis and backlinks to the originating note or source. Do not copy long derivations into the wiki unless the derivation itself is a durable reference.

### Code Experiment

Keep code in `code/`, outputs in `outputs/`, reusable plots and images in `figures/`, and link the experiment from related notes or wiki pages. Treat exploratory numerical output as exploratory until it has passed meaningful checks.

### Query And Synthesis

Answer questions by first deciding whether vault context matters. If it does, start from the newest relevant process-log entries and the index, then follow links outward. If an answer becomes reusable, file it back into the vault.

### Maintenance

Periodically check for orphan pages, missing links, duplicated concepts, stale summaries, and unfiled results. Maintenance should preserve creativity by keeping the structure navigable rather than imposing heavy labels.

## Provenance And Integrity

Use source links, backlinks, and page placement as the main provenance mechanism. Do not label every sentence.

Use explicit labels such as `speculative`, `open question`, `unchecked`, or `validated` only when the label changes how future agents should treat the content.

Do not present conjecture as derivation. Do not present heuristic reasoning as a validated result. Do not present numerical output as trustworthy without checks.

When relevant, distinguish between:

- what a source explicitly states;
- reconstructed intermediate steps;
- local interpretation;
- conjecture or open question.

For scientific or technical computing, record assumptions and conventions that materially affect the result. When possible, check limiting cases, symmetry constraints, dimensional analysis, normalization, Hermiticity or reality properties, convergence, or comparison to a toy analytic result.

## Error Handling

If confidence is limited, say why. Useful categories include:

- missing assumptions;
- conflicting conventions;
- incomplete derivation;
- questionable source material;
- numerical instability;
- inadequate convergence;
- possible implementation error;
- lack of a benchmark.

If a file is moved or renamed, update obvious incoming links when feasible. If a concept duplicates an existing page, merge or cross-link rather than silently creating parallel pages. If a source cannot be verified or a citation is unclear, mark the local note as `unchecked` or add a short uncertainty line.

If unsure whether to update durable wiki knowledge, prefer a compact process-log entry and a working note over rewriting stable wiki pages.

---
> Source: [zzhliu05/agent-formalizer](https://github.com/zzhliu05/agent-formalizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
