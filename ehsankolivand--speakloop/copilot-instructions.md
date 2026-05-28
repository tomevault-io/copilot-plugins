## speakloop

> <!-- SPECKIT START -->

<!-- SPECKIT START -->
Active feature: 006-feedback-quality-reliability — make the existing AI-derived
  feedback (grammar suggestions, cross-attempt narrative, single top-priority)
  reliably higher-quality. Same model family (Qwen3-8B); report format & schema_version
  stay 1; fully offline; no new feedback dimension (no ideal-answer judging). Grammar
  call site adopts vendor non-thinking config (temp 0.7) + repetition_penalty/stop +
  json-repair recovery (one new dep); narrative & top-priority STAY deterministic.
  Decisions: (1) single LLM call site, thinking off — no dual-mode; (2) stay 4-bit;
  8-bit OUT OF SCOPE this sprint (no A/B, no threshold; Constitution VI/VII); (3)
  json-repair now, Outlines deferred (its rep-penalty gap is RESOLVED — see research.md). New: eval/grammar/ held-out set +
  offline harness (baseline→post) proves SC-001/SC-002. Iterative: US1 reliability (MVP)
  → US2 grammar accuracy → US3 narrative + top-priority grounding.

Plan: specs/006-feedback-quality-reliability/plan.md
Spec: specs/006-feedback-quality-reliability/spec.md
Research: specs/006-feedback-quality-reliability/research.md (lifts doc/QWEN_IMPROVMENT_RESEARCH.md)
Data model: specs/006-feedback-quality-reliability/data-model.md
Contracts: specs/006-feedback-quality-reliability/contracts/ (grammar-output-schema · eval-set-format · report-invariance)
Code touchpoints: llm/qwen_engine.py (sampler + rep-penalty + stop), feedback/grammar_analyzer.py
  (json-repair + bounded regenerate; keep verbatim/coherence/no-op verification),
  feedback/narrative.py (tighten deterministic grounding); + eval/grammar/; pyproject.toml (+json-repair).

Prior feature: 005-context-engineering-audit — audited & rewrote the CLAUDE.md layer
  (root + 13 module files) as a code-true deliverable; launch footprint ≤ 6000 tokens.
  Plan: specs/005-context-engineering-audit/plan.md · Spec: specs/005-context-engineering-audit/spec.md

Prior feature: 004-public-release-readiness — cloneable & runnable by a stranger.
  Default questions ship in-repo at `content/questions.yaml`; `~/.speakloop/qa.yaml`
  is an opt-in override (precedence: --qa-file → home override → repo default).
  Adds a stdlib+git path-portability audit (pytest, < 2 s). No new dependency;
  report schema_version stays 1; MIT LICENSE present.
  Plan: specs/004-public-release-readiness/plan.md · Spec: specs/004-public-release-readiness/spec.md

Prior feature: 003-asr-l2-accent-accuracy — faithful transcripts on Persian-L1
  accented technical English. Default ASR Whisper-large-v3-turbo (mlx-whisper),
  Parakeet-TDT via `--asr-engine parakeet` + automatic fallback; per-session domain
  biasing + Silero-VAD; additive `asr:` frontmatter key (schema_version stays 1).
  Plan: specs/003-asr-l2-accent-accuracy/plan.md · Spec: specs/003-asr-l2-accent-accuracy/spec.md

Prior feature: 002-post-session-debrief — educational LLM grammar feedback
  (Persian-L1 catalog) + in-terminal interactive debrief.
  Plan: specs/002-post-session-debrief/plan.md · Spec: specs/002-post-session-debrief/spec.md
  New module: src/speakloop/debrief/ (render + audio + menu).

Base feature: speakloop v1 — local English interview-practice CLI.
  Plan: specs/001-v1-product-spec/plan.md · Spec: specs/001-v1-product-spec/spec.md

Engine selections cite the in-repo research documents:
  doc/research_tts.md (Kokoro-82M),
  doc/research_asr.md (Parakeet-TDT-0.6b-v3),
  doc/research_llm.md (Qwen3-8B 4-bit — deviates from the initial Qwen3.5-9B
    research choice because that HF repo turned out to be a VLM incompatible
    with mlx_lm.load(); see installer/manifest.py rationale comment).

Constitution: .specify/memory/constitution.md (v1.0.0).
Shipping order is three phases (A: listen-only, B: attempts + metrics, C: LLM feedback + trends);
each phase is a complete working system per Principle XII.
<!-- SPECKIT END -->

<!-- Human-authored map below. Anatomy (FR-010): overview · tech-stack · layout ·
commands · conventions · maintenance · traps · never-do · pointers. Code is the
source of truth; the constitution wins on any documentation conflict. -->

# speakloop — top-level map

## Overview

speakloop is a fully local, offline English speaking-practice CLI for non-native
software engineers preparing for technical interviews. It runs three local AI models
on Apple Silicon — TTS (Kokoro-82M), ASR (Whisper-large-v3-turbo), LLM (Qwen3-8B
4-bit) — to drive a listen → attempt (4/3/2) → feedback loop, written as Obsidian-
compatible Markdown reports. After the initial model download it makes zero network
calls (Constitution Principles II, III).

## Tech stack

Derived from `pyproject.toml`, confirmed against actual imports.

- **Python 3.12** — pinned `requires-python = ">=3.12,<3.13"` (`pyproject.toml:7`).
- **`uv`** — the only package manager (no `pip` workflows). Run everything via `uv run`.
- **CLI / UI**: `typer` (≥0.12), `rich` (≥13.7) — CLI only, no GUI (constitution constraint).
- **Config / data**: `pyyaml` (≥6.0, YAML user config), `python-frontmatter` (report YAML), `numpy` (≥1.26).
- **Audio**: `sounddevice` (≥0.4), `soundfile` (≥0.12).
- **Models / download**: `huggingface_hub` (≥0.24, resumable).
- **Engine packages** (each imported function-local in exactly ONE wrapper — Principle V):
  `mlx-whisper` (ASR), `parakeet-mlx` (ASR fallback), `silero-vad` (VAD), `mlx-lm` (LLM),
  `kokoro-mlx` (TTS).
- **`onnxruntime` (≥1.20)**: declared to pin the version, but **no direct import** in `src/` —
  it is transitive via `silero-vad` (divergence D-1).
- **`torchaudio<2.9`** (capped): ≥2.11 moves decoding to the unbundled `torchcodec` and crashes
  the first live VAD call — see Traps.

## Layout

Thirteen single-responsibility modules under `src/speakloop/`, each with its own `CLAUDE.md`
(Principles IV, XI). Dependency edges below are from an import scan (`rg "from speakloop\."`),
not prose (FR-007). Leaves: `config`, `content`, `trends`. Orchestrators: `cli` → 9 modules,
`sessions` → 6.

| Module | Responsibility | Depends on (internal) |
|--------|----------------|-----------------------|
| [`config/`](src/speakloop/config/CLAUDE.md) | Filesystem paths, Q&A-file resolution | — (leaf) |
| [`content/`](src/speakloop/content/CLAUDE.md) | Q&A YAML loader + schema (default `content/questions.yaml`) | — (leaf) |
| [`installer/`](src/speakloop/installer/CLAUDE.md) | Model manifest, consent, resumable download, validation | config |
| [`tts/`](src/speakloop/tts/CLAUDE.md) | TTS wrapper (**owns `kokoro_mlx`**) + clip cache | config, installer |
| [`audio/`](src/speakloop/audio/CLAUDE.md) | Playback, recording, device probing | sessions |
| [`asr/`](src/speakloop/asr/CLAUDE.md) | ASR wrapper (**owns `mlx_whisper`, `silero_vad`, `parakeet_mlx`**) + biasing | installer |
| [`llm/`](src/speakloop/llm/CLAUDE.md) | LLM wrapper (**owns `mlx_lm`** — Qwen3-8B 4-bit) | installer |
| [`metrics/`](src/speakloop/metrics/CLAUDE.md) | Per-attempt fluency metrics | asr |
| [`feedback/`](src/speakloop/feedback/CLAUDE.md) | Frontmatter, atomic writer, report builder, grammar analyzer | asr, config, llm |
| [`debrief/`](src/speakloop/debrief/CLAUDE.md) | Post-session interactive debrief (render + audio + menu) | feedback, tts |
| [`sessions/`](src/speakloop/sessions/CLAUDE.md) | 4/3/2 coordinator, timer, abort handling | asr, audio, config, content, feedback, metrics |
| [`trends/`](src/speakloop/trends/CLAUDE.md) | Cross-session dashboard | — (leaf) |
| [`cli/`](src/speakloop/cli/CLAUDE.md) | `typer` app: `practice`, `doctor`, `trends` | audio, config, content, feedback, installer, llm, sessions, trends, tts |

## Commands

Each verified by running it during this feature (see `specs/005-…/audit/command-matrix.md`).

```bash
uv run speakloop --help     # works with NO models downloaded (Principle VIII)
uv run speakloop doctor     # environment + model health check (exit 0 when healthy)
uv run pytest               # full suite — 363 passed, 3 skipped at HEAD (006 added eval + recovery tests)
uv run pytest -m live_asr   # real silero+torchaudio smoke — run when touching torchaudio
uv run pytest tests/integration/test_path_portability_audit.py   # no personal-path leakage
```

Linting uses `ruff` (config in `pyproject.toml [tool.ruff]`), but `ruff check .` currently
reports pre-existing findings on committed code, so it is **not** listed as a passing command
here (divergence D-7, deferred — fixing needs code edits).

## Conventions

Cross-verified against code and the constitution.

- **English-only** output everywhere (Principle I).
- **Engine imports are function-local** in their one wrapper file, so `--help` loads no models
  (Principle V + VIII).
- **Swappable engines**: replacing TTS/ASR/LLM touches exactly one wrapper file (Principle V).
- **`uv` only**; config is YAML; reports are Obsidian-compatible Markdown in `data/sessions/`
  named `YYYY-MM-DD-qXX.md` (Principle IX).
- **Report `schema_version` stays 1**; new frontmatter keys are additive only.
- **Conventional Commits** (`feat:`, `fix:`, `docs:`, …); every module ships its own `CLAUDE.md`.
- **Engine changes update the matching `doc/research_*.md`** (Principle X).

## Maintenance — how to keep this context layer true

Review is **feature-driven**: every new `specs/NNN-*` feature triggers this 7-item checklist
(≤ 2 minutes); plus any PR that changes a convention updates the relevant `CLAUDE.md` in the
same commit. No calendar cadence.

1. Re-read this file against the new feature's scope; update the SPECKIT block's active/prior
   lines and prune stale history (protects the ≤ 6000-token budget).
2. Run the documented commands (`--help`, `doctor`, `pytest`); remove any that now fail, add any new one.
3. For each module whose code changed (`git log --since=<last feature> -- src/speakloop/<mod>/`),
   re-check that module's `CLAUDE.md`.
4. Re-run the engine-import scan; confirm each engine package still resolves to exactly one wrapper (Principle V).
5. **Correct-twice-then-record**: if an agent is corrected on the same thing twice, add it here as a trap or never-do.
6. **PR-coupling**: any PR that changed a convention must have updated the relevant `CLAUDE.md` in the same commit — verify before merge.
7. Re-measure the launch footprint; if over 6000 tokens, push detail into a module file or a `paths`-scoped rule.

## Traps (evidence-cited)

1. **Don't bump `torchaudio` past `<2.9`** without `uv run pytest -m live_asr`: ≥2.11 moves
   decoding to unbundled `torchcodec`, crashing the first live VAD call (`pyproject.toml:29`,
   `asr/CLAUDE.md`, commit `21dfb86`).
2. **Keep engine imports function-local** — a module-level engine import would break `--help`.
   `tests/integration/test_help_without_models.py::test_importing_cli_loads_no_engine_packages`
   asserts importing the CLI loads none of `mlx_whisper`, `silero_vad`, `parakeet_mlx`, `mlx_lm`;
   `kokoro_mlx` is not yet covered by that guard (finding D-3).
3. **The LLM intentionally diverges from research**: `doc/research_llm.md` chose Qwen3.5-9B but
   that repo is a VLM incompatible with `mlx_lm.load()`; the code ships Qwen3-8B-4bit
   (`installer/manifest.py:56-65`). Don't "fix" it back.
4. **No personal absolute paths** (`/Users/...`) in any committed file — the path-portability
   audit fails CI otherwise (`tests/integration/test_path_portability_audit.py`, `specs/004-…`).
5. **Q&A precedence is `--qa-file → ~/.speakloop/qa.yaml → repo default`, no auto-copy** — the
   home file is opt-in, not created for you (`config/paths.py:103` `resolve_qa_file`, `specs/004-…`).
6. **Never bump report `schema_version`** — add frontmatter keys additively only
   (`feedback/frontmatter.py:20,40,91`, specs 002/003).
7. **Grammar JSON recovery is `json-repair`, not regex** — the old hand-rolled repair
   regexes (`_repair_json`/`_loads_lenient`/`json5`) were removed (006); don't reintroduce them.
   The Qwen generation config (temp 0.7, `repetition_penalty` 1.05/context 40, defensive
   `<|im_end|>` stop, `enable_thinking=False`, and the `retry=True` bump to 1.15/−0.1) lives
   **only** in `llm/qwen_engine.py`; the analyzer passes intent (`retry`), never engine config
   (`feedback/grammar_analyzer.py`, `specs/006-…`). 8-bit stays out of scope (Decision 2).

## Never do

- Add a network call after model download (Principle II) — no telemetry, no auto-update.
- Import an engine package (`mlx_whisper`/`silero_vad`/`parakeet_mlx`/`mlx_lm`/`kokoro_mlx`)
  at module top level, or from more than one file (Principle V; breaks `--help`).
- Edit `.specify/memory/constitution.md` from a normal feature (governance amendment only).
- Ship non-English user-facing strings (Principle I).
- Introduce a GUI, a non-YAML user config, or a `pip install` workflow (constitution constraints).

## Pointers

- Per-module guidance: each `src/speakloop/<module>/CLAUDE.md` (loaded on-demand).
- Specs: `specs/001`–`specs/005` (plan.md · spec.md per feature).
- Engine research: `doc/research_tts.md`, `doc/research_asr.md`, `doc/research_llm.md`,
  `doc/research_methodology.md`; context-engineering reference: `doc/research_context_engineering.md`.
- Governance: `.specify/memory/constitution.md` (v1.0.0) — wins on any conflict.

---
> Source: [ehsankolivand/speakloop](https://github.com/ehsankolivand/speakloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
