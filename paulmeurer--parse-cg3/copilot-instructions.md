## parse-cg3

> Common Lisp system for FST/CG morphosyntactic parsing of Georgian (all historical varieties: OG, MG, NG, JG) and Abkhaz. Two-phase pipeline: (1) morphological analysis via finite-state transducers, (2) disambiguation and dependency parsing via VISL CG-3 constraint grammars.

# CLAUDE.md — parse-cg3

## Project overview

Common Lisp system for FST/CG morphosyntactic parsing of Georgian (all historical varieties: OG, MG, NG, JG) and Abkhaz. Two-phase pipeline: (1) morphological analysis via finite-state transducers, (2) disambiguation and dependency parsing via VISL CG-3 constraint grammars.

## Loading

```lisp
;; Core system
(ql:quickload :parse-cg3)

;; Language-specific (load one):
(ql:quickload :parse-cg3-kat)       ;; Georgian with Xerox FST
(ql:quickload :parse-cg3-foma-kat)  ;; Georgian with foma
(ql:quickload :parse-cg3-abk)       ;; Abkhaz
```

Requires Clozure CCL v1.13, CG-3 (https://github.com/GrammarSoft/cg3), and either Xerox FST or foma for morphological transducers.

## Key files

- `parse-text.lisp` — Core `parsed-text` class, `parse-text` entry points, two-phase `process-text` pipeline, CoNLL-U export
- `cg3-disambiguate.lisp` — CG3 grammar loading, `split-grammar-file`, `cg3-disambiguate-text`
- `parse-api.lisp` — REST/JSON API server
- `parse-kat.lisp` — Georgian: FST init, morphological lookup, UD mapping
- `parse-abk.lisp` — Abkhaz: coordination compounds, morphological lookup, UD mapping
- `cg3/` — CG3 grammars: `*-dis.cg3` (disambiguation), `*-dep.cg3` (dependency)

## CG3 grammars

Grammar files in `cg3/` use VISL CG-3 syntax. Georgian grammars (`geo-dis.cg3`, `geo-dep.cg3`) are split at load time into variety-specific files (og/mg/ng) using variety prefixes. Abkhaz grammar (`abk-dis.cg3`) handles both disambiguation and dependency in a single file.

## Coding conventions

- Lisp documentation: explain "why" not "what"; use `;;;;` for major section headers and `;; ====` dividers; document contracts for exported functions; flag subtleties
- CG3 documentation: section-level comments explaining design rationale and linguistic patterns; include corpus examples where helpful
- Packages: core code in `PARSE` package (nicknamed `KP`), FST interface in `FST` package

## Dependencies

`:cl-fst` (or `:cl-foma`), `:cl-vislcg3`, `:aserve`, `:graph`, `:st-json`, `:framework-1-2`

## Gold corpora

- Georgian UD treebank at `~/quicklisp/local-projects/ud/UD_Georgian-GNC/` (CoNLL-U format)
- Abkhaz UD treebank at `~/quicklisp/local-projects/ud/UD_Abkhaz-AbNC/` (CoNLL-U format)

---
> Source: [paulmeurer/parse-cg3](https://github.com/paulmeurer/parse-cg3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
