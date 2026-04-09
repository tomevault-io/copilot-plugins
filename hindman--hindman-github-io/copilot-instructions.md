## hindman-github-io

> The Fifth Fret is a personal Jekyll/GitHub Pages site at hindman.github.io.

# Project: hindman.github.io (The Fifth Fret)

## The Website

The Fifth Fret is a personal Jekyll/GitHub Pages site at hindman.github.io.
Topics: guitar technique, music theory, fingerstyle, blues, US politics,
economics, and occasional other interests. Built with Minimal Mistakes theme.
Audience: personal use plus a small guitar-learning community.

Local dev: `bundle exec jekyll serve` then open http://127.0.0.1:4000/

## LoopLlama

A browser-based YouTube video controller for music practice, hosted at
hindman.github.io/loopllama/. Core use case: hold a guitar and control
playback via Vim-style key bindings -- loop sections, set marks, adjust
speed, navigate quickly.

- v1: single vanilla JS file (`loopllama/v1/loopllama.js`) + YouTube
  IFrame API. Flat, procedural, global-variable codebase. Uses browser
  `prompt()` dialogs for all user input.
- v2: in planning. Clean rewrite at `loopllama/v2/` using Lit + Vite.

## LoopLlama v2 Status

Active planning phase as of February 2026. The full design plan is at:

    _notes/v2-planning/loopllama-v2-plan.md

Read that file before working on v2. It covers: goals, tech decisions
(Lit, Vite, Shoelace), data schema, sections model, looping model,
directory structure, and implementation stages.

Key v2 decisions in brief:
- Three first-class entities: Sections (musical structure, divider-based),
  Loops (named loopable ranges), Marks (named time points).
- Active loop is always a scratch loop; named entities are untouched until
  explicitly saved back.
- Keyboard-first (Vim-style multi-key bindings) plus full mouse support.
- No browser prompt() dialogs; Shoelace components for UI primitives.
- Favorites concept dropped; replaced by named/labeled videos with picker.
- Storage: localStorage only; no backend for v2.

## Notes

- `_notes/v2-planning/schematic-mockup.txt` has a UI mockup and tentative
  key binding sketches.
- `_notes/project-notes.md` has general dev notes and TODOs.
- Git is read-only per global CLAUDE.md: do not commit, push, or stage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hindman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hindman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
