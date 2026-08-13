## tugamorph

> Rule-based morphological analyzer for Portuguese. Segments words into prefixes, roots, suffixes, verbal inflections, and clitics, and emits structured analyses (dataclass, feature dict, 17-dim vector, JSON) for NLP pipelines and linguistic research.

# AGENTS.md — tugamorph

Rule-based morphological analyzer for Portuguese. Segments words into prefixes, roots, suffixes, verbal inflections, and clitics, and emits structured analyses (dataclass, feature dict, 17-dim vector, JSON) for NLP pipelines and linguistic research.

## Setup

```bash
pip install -e .
```

Pure standard library (Python 3.8+); no required runtime dependencies. Optional integrations improve accuracy if importable:

```bash
pip install git+https://github.com/TigreGotico/silabificador   # syllabification
pip install git+https://github.com/TigreGotico/tugatagger       # POS disambiguation
```

## Test

```bash
python -m unittest discover tests -v
```

~250 tests in `tests/test_pt_morphology.py` covering character features, prefix/suffix extraction, the full verbal paradigm, irregular verbs, clitics (enclitic + mesoclitic), compounds, phonology, feature export, POS disambiguation, and regression cases.

## Lint/Typecheck

None configured.

## Layout

- `tugamorph/__init__.py` — the entire package in one module (~1740 lines): enums (`MorphemeType`, `PrefixCategory`, `SuffixCategory`, `StressPattern`, `CliticPosition`), dataclasses (`Morpheme`, `CliticInfo`, `VerbalAnalysis`, `PhonologicalFeatures`, `MorphologicalAnalysis`, `AnalysisConfig`), the lexical data tables (`PREFIX_TABLE`, `SUFFIX_TABLE`, `IRREGULAR_STEMS`, irregular whole-word table, clitic table), and the `PortugueseMorphAnalyzer` orchestrator.
- `tugamorph/version.py` — semver block managed by gh-automations.
- `tests/test_pt_morphology.py` — single unittest module.
- `setup.py` — packaging.

Entry points: none. This is an importable library, not an OVOS/OPM plugin or skill — no entry-point group is declared.

Public API: `PortugueseMorphAnalyzer`, `AnalysisConfig`, plus the module-level data tables which are intended to be extended in place (`PREFIX_TABLE.append(...)`, `analyzer._irregular_whole_words[...]`, `analyzer._prefix_block_stems[...]`).

## Conventions (Org hard rules)

- Branches: `dev` (work) and `master` (stable). NEVER `main`.
- Never edit `tugamorph/version.py`. gh-automations bumps semver from conventional-commit prefixes (`feat:`, `fix:`, `feat!:`).
- New repos private by default; do not make a source repo public without asking.
- Commit identity: JarbasAi <jarbasai@mailfence.com>.
- Reference `OpenVoiceOS/gh-automations` reusable workflows at `@dev`.
- No Neon / `neon-*` references.
- No meta-commentary in docs/commits/code (no history, no dates, describe current state only).
- CI is provided by OpenVoiceOS/gh-automations reusable workflows.

## Gotchas

- The package is a single large `__init__.py`; all logic and data live there. Editing the lexical tables is the normal extension path.
- Analysis is single-best-parse and heuristic — no ambiguity modeling, no full lemmatization (only `lemma_guess` for irregular verbs). The pipeline is ordered: irregular whole-word lookup short-circuits first, then clitics, compounds, prefix stacking, irregular stems, suffix vs verbal ending (longest-match-wins), root extraction, phonology.
- Suffix-vs-verbal ties: longest match wins; equal-length ties default to verbal. A POS tagger (tugatagger), when present, overrides this via `pos_disambiguate`.
- The prefix blocklist (`_prefix_block_stems`) is manually curated; uncommon words may get false prefix splits.
- Phonology is estimated from orthography unless silabificador is installed.
- European Portuguese focus; BP-specific morphology is not specially annotated.

---
> Source: [TigreGotico/tugamorph](https://github.com/TigreGotico/tugamorph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
