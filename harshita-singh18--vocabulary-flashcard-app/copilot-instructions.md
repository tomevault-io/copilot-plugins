## vocabulary-flashcard-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A vocabulary flashcard web app built with Flask. Users can add words and definitions, then quiz themselves with flashcards.

## Stack

- Python / Flask
- Jinja2 templates, vanilla HTML/CSS
- pytest

## Rules

- Use Flask for all web functionality — no other frameworks.
- Store flashcards as a list of dicts in memory (e.g. `{"word": "...", "definition": "..."}`). No database unless explicitly agreed.
- Do not add new dependencies without asking first.
- Do not modify test files to make tests pass — fix the app code instead.

---
> Source: [harshita-singh18/vocabulary-flashcard-app](https://github.com/harshita-singh18/vocabulary-flashcard-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
