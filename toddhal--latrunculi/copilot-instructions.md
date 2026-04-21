## latrunculi

> *Read this first. Every session. Every agent.*

# CLAUDE.md — Latrunculi Master Briefing
*Read this first. Every session. Every agent.*

## What This Project Is
Converting latrunculi_full.html into a React web app, then eventually
a React Native iOS app. This is a learning project for Todd.
Explain your reasoning as you work. Don't just do — teach.

## The Source of Truth
- latrunculi_full.html — the complete working game. READ ONLY. Never modify it.
- REACT_ROADMAP.md — the conversion plan. Follow the steps in order.
- TASKS.md — the current active task. Start here each session.
- PROGRESS_TRACKER.md — update checkboxes as work is completed.

## The Agent Team
When given a multi-step task, spawn specialized subagents:

- architect-agent: reads the HTML file, maps the React component structure
- engine-agent: extracts pure JS logic into src/engine/ files
- component-agent: builds React components one at a time
- testing-agent: writes and runs tests after each component
- review-agent: checks that the Roman aesthetic is preserved

## The Rules
1. Engine files must be pure JS — no React, no DOM references.
2. Follow REACT_ROADMAP.md step order. Don't skip ahead.
3. One step fully complete before the next begins.
4. Update PROGRESS_TRACKER.md checkboxes as things are completed.
5. The Roman aesthetic (Cinzel font, dark stone, gold/crimson) must be preserved.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toddhal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
