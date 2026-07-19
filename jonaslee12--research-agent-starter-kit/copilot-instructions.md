## research-agent-starter-kit

> <!-- Customise this file with your own project details -->

<!-- Customise this file with your own project details -->

# Research Project Agent Guide

Project: `PROJECT_TITLE`

This repository is a reusable research-agent starter kit. It can be adapted for dissertations, theses, journal articles, grants, fieldwork, policy reports, design research, evidence synthesis, and knowledge-base projects.

Replace placeholders with project-specific facts only after checking a source file or receiving direct user confirmation.

## Project Profile

Before serious work, choose a profile in `RESEARCH_PROJECT_BRIEF.md` using `PROJECT_TYPE_PROFILES.md`.

Default profile: `TO CONFIRM`

Examples:

- taught dissertation / thesis
- journal article / manuscript
- research proposal / grant
- qualitative fieldwork project
- quantitative or computational study
- design research / product research
- policy / practice report
- literature review / evidence synthesis
- knowledge-base / RAG project

## Project Skills

Project-level skills live in `.agents/skills/`.

Use these skills for research-project work. Some skill names still begin with `dissertation-*` for backwards compatibility; treat them as general research workflows unless the selected project profile is specifically a dissertation or thesis.

- `agent-orchestration`: route each task to the right skills and decide whether subagents are useful.
- `dissertation-source-first-gate`: check source files before drafting formal text or factual claims.
- `dissertation-document-quality-gate`: review formal outputs before delivery.
- `dissertation-learning-loop`: turn new reading and confirmed decisions into durable project memory.
- `dissertation-literature-review`: plan and synthesise literature review work.
- `dissertation-research-search-protocol`: structure literature, web, policy, LMS, and source searches.
- `cognitive-frameworks`: make section type, claim, gap/problem type, evidence, warrant, boundary, and rhetorical plan explicit before major writing.
- `academic-self-review-loop`: run a two-pass writing-quality review and revision loop before style and document-quality gates.
- `academic-integrity-preflight`: check prompt residue, placeholders, fake references, unsupported claims, unresolved compliance requirements, and AI-use disclosure boundaries before formal drafting or delivery.
- `authorial-voice-integrity`: handle AI-writing, de-AI, humanising, detector-framed, generic-AI-style, and AI-use disclosure writing requests by improving authorial voice and integrity without detector-evasion.
- `style-fingerprint-gate`: scan repeated binary negative-contrast templates such as `rather than`, `not...but`, `不是...而是`, and `而不是` before formal delivery.
- `material-passport`: package source readiness, compliance or requirement evidence, citation boundaries, and open confirmations before formal writing or delivery.
- `formal-delivery-guard`: create/check pre-delivery locks and final guard reports before presenting formal artifacts as usable.
- `dissertation-argument-spine`: build the controlling argument and section logic.
- `dissertation-chapter-plan`: plan chapters, section jobs, and writing schedules.
- `dissertation-research-review`: review research design, questions, methods, claims, and drafts.
- `dissertation-citation-audit`: verify citations and claim support.
- `uk-academic-writing-style`: check British-English academic style when relevant.
- `style-memory-and-revision-gate`: apply user style preferences and prohibited-phrase checks.
- `context-continuity`: keep task state and handoff notes usable across long work.
- `dissertation-knowledge-ops`: maintain research-wiki, knowledge-base, Obsidian, and source registers.
- `dissertation-agent-self-debug`: diagnose false runs, stale assumptions, or shallow checking.
- `dissertation-agent-architecture-audit`: audit rule stacks, memory layers, and skill routing.
- `dissertation-workspace-surface-audit`: audit local tools, files, rendering, connectors, and missing surfaces.
- `release-surface-verification`: verify GitHub Releases, About/sidebar, topics, rendered README/docs, and public links before claiming a public release or template update is complete.
- `dissertation-automation-audit`: audit scheduled checks, hooks, monitors, and automation safety.
- `dissertation-skill-stocktake`: review skills for overlap, stale rules, and trigger clarity.
- `brainstorming`: structure unclear research-project or agent-system ideas before drafting, implementation, or skill changes.
- `project-skill-creator-governance`: govern new or updated project skills and route SKILL.md authoring to the global `skill-creator` skill.
- `playwright-dissertation-browser`: safely route browser automation to the global `playwright` skill while preserving read-only and privacy boundaries.
- `markitdown`: guide file-to-Markdown conversion for source review, literature ingestion, Obsidian notes, or RAG-ready knowledge-base preparation.
- `research-project-adapter`: map the starter kit to the selected project profile and decide which dissertation-specific files are optional.
- `research-neural-network-figure`: plan or audit neural-network architecture figures and tool routes such as NN-SVG, PlotNeuralNet, draw_convnet, TikZ, or custom SVG.
- `research-nature-figure`: apply high-impact scientific figure-contract logic to data, conceptual, architecture, and multi-panel figures.
- `research-nature-writing`: sharpen high-impact article-style argument structure after source, compliance, citation, cognitive, and self-review gates.
- `using-superpowers`: apply a project-safe Superpowers-style skill-first workflow without overriding source-first, quality-gate, privacy, or window-separation rules.

Global/system skills intentionally used by this template:

- `skill-creator`: use with `project-skill-creator-governance` when creating or updating `SKILL.md` files.
- `playwright`: use with `playwright-dissertation-browser` when a task needs CLI browser automation.

Domain-specific skills are included as optional examples. Rename, edit, or remove them if your research topic does not need them.

Archived project skills are kept under `.agents/skills/_archived/` and must not be routed as active skills unless Maintenance restores them. v1.8.0 moves topic-specific or late-phase example packs out of the default active set to reduce context load: `active-learning-design-support`, `ai-agent-design-spec`, `codesign-output-synthesis`, `dissertation-figure-spec`, `dissertation-research-wiki`, `prototype-evaluation-audit`, `teacher-adoption-modeling`, `teaching-knowledge-base-plan`, and `viva-prep`. Restore one only when the current project phase has a concrete recurring need for it.

## Safety Rules

- Treat `USER_DASHBOARD.md` as the user-facing status hub if you create one.
- For non-trivial tasks, first use `agent-orchestration` to classify the task and select skills.
- When the user knows the goal but not the workflow, use `docs/TASK_CARDS.md` or `docs/TASK_CARDS_CN.md` to choose the smallest safe route, then use `templates/SOURCE_FIRST_INTAKE_CARD.md` to collect missing source, output, evidence, privacy, and gate fields before acting.
- For project-type setup, first read `PROJECT_TYPE_PROFILES.md` and `RESEARCH_PROJECT_BRIEF_TEMPLATE.md`.
- For substantial Production or Maintenance tasks, run deterministic runtime preflight first when local tools are available:
  - `python3 scripts/agent_runtime.py "<TASK>" --window Production --write --strict`
  - or `python3 scripts/agent_runtime.py "<TASK>" --window Maintenance --write --strict`
  If it returns `BLOCKED`, fix the missing file or gate before continuing.
- Keep bounded tasks bounded. Source-planning, literature-priority sorting, source lookup, and minor citation/typo edits should use light receipt sets unless the user asks for formal prose, Word/DOCX, stakeholder-facing or submission-facing output, or protected source-of-record edits. Methodology/literature keywords alone do not justify the full formal-writing chain.
- Use `scripts/session_log_integrity_check.py --strict` when auditing runtime/session records. Malformed JSONL, illegal window labels, runtime/window mismatches, or unpaired `session_start` events are maintenance issues, not harmless log noise.
- For Codex `logs_*.sqlite` / WAL growth, use `scripts/codex_sqlite_log_guard.py` before suggesting cleanup. Start with read-only `scan` or `monitor`. Do not delete, move, checkpoint, or add SQLite triggers to Codex log files unless the user explicitly asks for remediation, Codex is fully closed or the user accepts an override, and the target is confirmed as a Codex diagnostic `logs_*.sqlite` database rather than state, memory, goal, session, or project data.
- For long sessions, context compression, or "the agent suddenly got worse" symptoms, use `scripts/context_health_signal.py` to record a lightweight context-health signal. Runtime preflight writes an automatic route signal when `--write` is used, but actual turn count, exact token count, compression notices, and model routing may need manual entry because most agent hosts do not expose them to local scripts.
- Treat `.codexignore` as a context-load guard, not a privacy or publication control. It keeps receipts, runtime state, audit reports, logs, and generated documents out of default agent context; still run `scripts/privacy_check.sh` before public sync.
- For long-running projects, use `research-wiki/STAGE_GRAPH.md` and `research-wiki/STAGE_CONTINUITY_PROTOCOL.md` before a later-stage task changes, designs, restates, translates, formalises, or produces a high-risk deliverable with upstream dependencies. High-risk deliverables include proposal/brief material, method plans, compliance or ethics material, fieldwork instruments, concept cards/scenario stimuli, RQ-to-method mapping, analysis plans, stakeholder-facing decision memos, and formal chapter/section drafts.
- Use the runtime `recall_decision` or run `python3 scripts/stage_recall_policy.py --task "<TASK>"` as the Token-Aware Recall controller. Tier 0=no project recall, Tier 1=anchor scan, Tier 2=pointer lookup, Tier 3=targeted Stage Continuity Capsule, Tier 4=full upstream audit or pause. This controller saves context but cannot override source-first, compliance, citation, privacy, document-quality, delivery, or Stage Continuity A+B gates.
- If a user asks to skip an upstream check for a triggered deliverable, surface the omitted dependency first. Only after explicit user acceptance may the task proceed as an override risk; do not call the Stage Continuity Gate a pass.
- For non-obvious route, method, instrument, analysis, or delivery decisions, write a concise Deep Reasoning Pass before drafting: decision under consideration, chosen direction and concrete trade-off accepted, rejected alternative only if genuinely considered, and what would change the decision. Do not expose private chain-of-thought.
- Do not invent names, emails, supervisor/PI/client details, dates, funder/journal/client/institutional requirements, rubrics, citations, participant facts, datasets, results, or findings.
- For formal drafting or editing, use `dissertation-source-first-gate`.
- For substantial proposal, manuscript, report, grant, literature review, methodology, or stakeholder-facing writing, use source-first, then `material-passport`, then `academic-integrity-preflight`, then `cognitive-frameworks` before drafting.
- For formal proposal, thesis/dissertation, manuscript, report, grant, stakeholder-facing argument claims, citation claim-support audits, or source-readiness upgrades, use `research-wiki/CLAIM_LEDGER_LITE_PROTOCOL.md`. Run `scripts/claim_ledger_lite_check.py <ledger.md>` when local tools are available. Claim Ledger Lite constrains claim strength; it does not make metadata-only sources citation-ready and it does not replace source-section verification.
- For formal academic or professional prose, use `academic-self-review-loop` after cognitive planning and before style/document-quality gates.
- For formal-writing pipelines, use `material-passport` after source-first checks and before the artifact moves from planning to drafting, drafting to review, or review to delivery. Use a short passport for internal movement and a full passport for reviewer/stakeholder/submission-facing outputs.
- For important Word, PDF, or stakeholder-facing delivery, follow `research-wiki/DOCUMENT_PIPELINE.md` and record thinking, writing, and delivery checkpoints, or mark delivery checkpoint not applicable.
- For rubric, marking criteria, journal author guidelines, funder rules, client requirements, deadlines, word counts, or submission rules, use the strongest available project requirement source. For assessed academic work, use `university-guidance/RUBRIC_EVIDENCE_GATE.md`.
- For formal, supervisor/PI/client/reviewer-facing, or submission-facing documents, use `quality-gates/PROJECT_DELIVERY_REVIEW_GATE.md` before delivery. Use `university-guidance/DISTINCTION_DELIVERY_REVIEW_GATE.md` only when the selected profile is assessed academic work and a high-band target is relevant.
- For formal outputs, use `dissertation-document-quality-gate` before delivery.
- For Word/PDF, figure, diagram, browser, Obsidian, public GitHub, or other user-visible deliverables, use `research-wiki/VISIBLE_OUTPUT_QA_PROTOCOL.md`: define the artifact's communication job, render or open the visible output, run deterministic checks, visually inspect the result, compare against a previous accepted baseline when available, then record a delivery verdict. Use `scripts/visible_output_qa_check.py <qa-note.md>` for substantial visible-output handoffs. A visible-output pass does not prove citation support, compliance readiness, academic/professional quality, or submission readiness.
- For formal, reviewer-facing, stakeholder-facing, or submission-facing prose, run `style-fingerprint-gate` and `scripts/style_fingerprint_scan.py` after self-review/authorial voice work and before delivery. Treat it as a writing-quality scan, not an AI detector.
- For substantial formal tasks, use `scripts/agent_runtime.py` to read the task-specific `receipt_requirements`. Required skills should create execution receipts with `scripts/skill_execution_receipt.py`; missing required receipts are delivery-blocking unless the user explicitly records an override.
- Before presenting a formal Word, PDF, reviewer-facing, stakeholder-facing, client-facing, or submission-facing artifact as usable, use `formal-delivery-guard` with `scripts/pre_delivery_lock.py` and `scripts/formal_delivery_guard.py`. For Markdown-to-Word outputs, run `scripts/markdown_docx_structure_check.py` through the guard or directly before delivery. Missing Word tables, residual pipe-delimited table paragraphs, or silent loss of previously accepted Word structures are delivery-blocking unless the user explicitly records a risk override.
- For important formal `.docx` outputs, run `scripts/docx_layout_review_check.py` and record a layout self-review verdict. When a previous accepted `.docx` exists, pass it as `--previous-docx`. Deterministic checks catch table, heading, list, and visible hierarchy regressions, but do not replace visual inspection of rendered pages. Any `--skip-*` or `--allow-*` DOCX layout exception requires `--layout-decision-reason`.
- For academic or professional prose, use `uk-academic-writing-style` and `style-memory-and-revision-gate` when language quality matters. Adapt spelling, tone, and format to the project context.
- For AI-writing, de-AI, humanising, lower-AI-rate, AIGC, AI detector, or disclosure-hiding requests, use `authorial-voice-integrity` and `research-wiki/AI_WRITING_AUTHORIAL_VOICE_POLICY.md`. Reframe the task as authorial voice, academic/professional integrity, and evidence-led style. Do not promise detector scores, use evasion tactics, or hide AI-use disclosure.
- Do not route to archived skills by default. If a task appears to need an archived topic pack, first use the active core workflow (`research-project-adapter`, `brainstorming`, `cognitive-frameworks`, source-first, compliance/privacy, document-quality, and relevant review skills). Restore the archived skill only after Maintenance records why the phase needs it.
- When adapting public style or workflow patterns, run `scripts/borrowed_pattern_boundary_lint.py` when local tools are available. Boundary language is allowed, but positive instructions that promise detector evasion, detector-score optimisation, authorship verdicts, or humanising-as-evasion are not allowed.
- Keep confirmed evidence separate from interpretation.
- Mark unresolved facts as `TO CONFIRM`.
- Treat participant data, identifiable notes, recordings, transcripts, and signed consent forms as sensitive.
- Keep raw participant data outside this repository unless it is ethically approved, anonymised, and intentionally stored.
- Use `compliance/PROJECT_COMPLIANCE_TRACKER.md` before handling ethics, IRB, privacy, funder, journal, client, IP, AI-use, or data-management requirements.
- When editing official templates, preserve fixed template text unless clearly editable.
- Before GitHub sharing, run `scripts/privacy_check.sh` and complete `PRIVACY_CHECKLIST.md`.
- When preparing public commits, stage intended files explicitly by path. Do not use `git add -A` for release or template updates.
- Before syncing private-workspace improvements into this public starter kit, read `PUBLIC_SYNC_POLICY.md` and separate shared-core improvements from private-only project material.
- Before claiming a GitHub release, public template update, version bump, About/sidebar change, topic update, README badge change, or public documentation update is complete, use `release-surface-verification`. Check the user-visible GitHub surface, not only local files, commits, branches, or tags.
- Use `brainstorming` before high-impact or unclear research route, method, concept-card, skill, or system design decisions when the next action is not obvious.
- Use `project-skill-creator-governance` before adding, copying, adapting, or updating project skills; use global `skill-creator` for SKILL.md authoring.
- Use `playwright-dissertation-browser` before browser automation for LMS, local previews, or browser-visible verification; do not submit, download, or modify browser content without explicit confirmation.
- Use `markitdown` before file-to-Markdown conversion; check whether MarkItDown is installed and do not install it without explicit confirmation.
- Use `scripts/academic_database_connector.py` for academic metadata searches when available. Treat all search results as `METADATA ONLY` until source sections are reviewed.
- Use `scripts/citation_style_check.py` and `scripts/citation_claim_audit.py` for citation-heavy drafts when available. Citation consistency is not proof of claim support.
- Use `scripts/claim_ledger_lite_check.py` when formal claims or source-readiness changes need a claim ledger. A ledger pass is a structure/boundary check, not source-support proof.
- Use `academic-integrity-preflight` and `scripts/academic_integrity_preflight.py` before substantive formal drafting and again before final delivery. This is not an AI detector.
- Use `scripts/authorial_voice_scan.py` when prose has generic AI-style, detector-framed, or disclosure-risk wording. The scan is advisory for authorial voice and integrity; it is not an AI detector.
- Use `scripts/style_fingerprint_scan.py` when formal prose risks repeated binary contrast phrasing. Keep legitimate scope distinctions; remove mechanical repetition.
- Use `scripts/skill_execution_receipt.py` after required gates produce evidence. Receipts prove execution evidence exists; they do not prove academic sufficiency or source support.
- Use `scripts/material_passport.py` to create a readiness passport for formal artifacts. It packages evidence status; it does not prove claim support, approval, acceptance, or official compliance.
- Use `scripts/pre_delivery_lock.py` and `scripts/formal_delivery_guard.py` before final formal delivery. For `.docx` outputs, the guard should also run Markdown-DOCX structural parity and DOCX layout review unless the user explicitly accepts and records a delivery risk. Overrides are allowed only with explicit user acknowledgement and do not become quality passes.
- Use `scripts/visible_output_qa_check.py` before claiming that a visible output surface has been checked. Commits, local files, tags, or generated documents are not enough when the failure mode is visible to a reader.
- Use `scripts/codex_sqlite_log_guard.py scan` or `monitor` before diagnosing Codex diagnostic-log write growth. Trigger installation, WAL checkpointing, and old-log archiving are dry-run by default and require explicit `--apply`; write actions should also require `--confirm-codex-closed` unless the user deliberately accepts the risk.
- Use `scripts/claude_independent_review.py` only for optional independent review of safe, non-sensitive artifacts. Claude Code feedback is advisory; it does not replace local source, privacy, citation, compliance, or delivery gates.
- Use `knowledge-base/self-growing/` for controlled knowledge-base growth. New items must pass source, evidence-status, privacy, and destination checks before they move from `raw-inbox/` into compiled wiki or formal sources.
- Use `scripts/build_agent_index.py`, `scripts/local_retrieval_search.py`, `scripts/build_vector_index.py`, and `scripts/vector_retrieval_smoke_test.py` only as local retrieval aids. Retrieval finds candidate files; it does not prove claim support.
- Use `research-neural-network-figure` before planning actual neural-network/model architecture visuals. Do not install or run third-party figure repositories without explicit confirmation.
- Use `research-nature-figure` before high-impact or multi-panel research figures. It is a figure-quality layer, not a source/data validity layer.
- Use `research-nature-writing` only after required evidence and integrity gates for formal prose. It can sharpen article-style logic but must not inflate claims or invent citations.
- Use `research-wiki/GENERAL_RESEARCH_SKILL_COMPATIBILITY_CONTRACT.md` before exporting local `research-*` skills into this public starter kit.
- Use `docs/WEEKLY_LITERATURE_GAP_WATCH_AUTOMATION.md` before creating or revising a weekly literature-monitoring automation. Preserve the candidate-only boundary unless the user explicitly confirms ingestion.

## Deliverable Preferences

- Use Word `.docx` for formal outputs when possible.
- Use Markdown for internal project memory, checklists, source maps, logs, and notes.
- Preserve original files and create revised copies unless overwrite is explicitly requested.
- Label draft status clearly: working draft, review draft, supervisor/PI/client draft, or submission/client-ready after user confirmation.

## Recommended Workflow

1. Route the task with `agent-orchestration`.
2. If route or input fields are unclear, use `docs/TASK_CARDS.md` and `templates/SOURCE_FIRST_INTAKE_CARD.md` before proceeding. Task Cards and intake cards are planning aids; a completed intake is not evidence, citation readiness, or source-section verification.
3. Run `scripts/agent_runtime.py` for substantial tasks when local tools are available.
4. Read `RESEARCH_PROJECT_BRIEF.md` if present; otherwise use `RESEARCH_PROJECT_BRIEF_TEMPLATE.md` and mark project facts `TO CONFIRM`.
5. Read `PROJECT_AGENT_PREFERENCES.md` and relevant task-state files.
6. Compute or consume the Token-Aware Recall tier; apply Stage Continuity A+B when triggered.
7. Use source-first checks before formal writing.
8. Use `material-passport` to package source, compliance/requirement, citation, and `TO CONFIRM` status before formal artifacts move forward.
9. Use `academic-integrity-preflight` before major revision and again before delivery.
10. Use `cognitive-frameworks` before major argument, gap, methodology, literature, proposal, manuscript, report, grant, or stakeholder-facing drafting.
11. Use `academic-self-review-loop` before style polishing and document-quality review for formal prose.
12. Use `authorial-voice-integrity` when the task involves AI-style prose, humanising, de-AI, detector framing, or AI-use disclosure.
13. Use `style-fingerprint-gate` for formal prose before delivery when repeated contrast templates could become visible.
14. Record required skill execution receipts for substantial formal tasks.
15. Use the learning loop after useful reading or confirmed decisions.
16. Use `knowledge-base/self-growing/` for controlled intake, growth queue triage, and compiled-wiki navigation.
17. Use source-readiness checks before citation-heavy writing.
18. Use compliance checks before ethics, privacy, funder, journal, client, or data-management claims.
19. Use rubric or requirement evidence checks before grade-band, journal, funder, deadline, or word-count claims.
20. Use `research-wiki/DOCUMENT_PIPELINE.md` for important Word/PDF/stakeholder-facing delivery.
21. Use the project delivery review gate before formal document delivery.
22. Use `formal-delivery-guard` before presenting formal artifacts as usable.
23. Use relevant academic/professional style gates before delivering prose.
24. Use document-quality gate before delivering formal outputs.
25. Update `research-wiki/TASK_STATE.md` after substantial work.
26. Record substantial Production work in `research-wiki/PRODUCTION_RUN_REGISTER.md` if that register is enabled.
27. Use `brainstorming` for unclear, high-impact route decisions before drafting or system changes.
28. Use `project-skill-creator-governance` and global `skill-creator` before adding or changing skills.
29. Use `playwright-dissertation-browser` and global `playwright` for controlled browser automation.
30. Use `markitdown` only after checking tool availability and privacy boundaries.
31. Use `research-*` figure/writing skills only as optional quality layers after source, privacy, compliance, citation, and document gates.
32. Use `scripts/claude_independent_review.py` for optional context-naive independent review when the artifact is safe to send to Claude Code.
33. Use staged literature gap-watch automation only for candidate discovery unless the user confirms ingestion.
34. Use `release-surface-verification` before saying a public GitHub release or template update is visible and ready for readers.
35. For Codex `logs_*.sqlite` / WAL growth, diagnose with `scripts/codex_sqlite_log_guard.py scan` or `monitor` first; only apply trigger/checkpoint/archive actions after confirming the target and write-safety boundary.

## Public Template Boundary

This repository should remain generic. Do not commit:

- personal dissertation drafts;
- private supervisor, PI, reviewer, client, or funder feedback;
- LMS, intranet, journal portal, funder portal, client portal, or restricted content;
- raw participant data;
- signed consent forms;
- API keys, tokens, cookies, or browser profiles.

---
> Source: [JonasLee12/research-agent-starter-kit](https://github.com/JonasLee12/research-agent-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
