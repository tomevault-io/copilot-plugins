## llm-wiki-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A distribution pack of five Claude Code skills that together implement a personal wiki pattern on top of an Obsidian vault. There is no build, no test suite, no runtime — the "code" is skill markdown files that Claude Code loads and executes. Editing this repo means editing skill prompts and the shipped contract template.

## Architecture

Three layers the skills operate on in a target workspace (not this repo):

- **Raw/** — immutable source clippings (Obsidian Web Clipper output). Read-only to the LLM. New items are marked `compiled: false` in frontmatter; `wiki-compile` flips them to `true` after ingest.
- **Wiki/** — LLM-owned synthesized articles plus `index.md` (catalog) and sibling `Raw/log.md` (compile run history).
- **The contract** — writing rules, frontmatter spec, structure. Lives as `skills/wiki-init/assets/contract-template.md` in this repo. `wiki-init` inlines it into the target's `CLAUDE.md` as a `## Wiki` section, stripping the top `# Wiki` header and demoting every remaining heading one level so it nests cleanly. The runtime skills (`compile`, `search`, `save`, `lint`) then read the target's `CLAUDE.md` for rules — there is no separate `Wiki.md` at the target root.

Skill relationships:

- `wiki-init` — one-shot bootstrap. Creates `Raw/` + `Wiki/`, inlines the contract into `CLAUDE.md`. Idempotent; never silently overwrites an existing `## Wiki` section.
- `wiki-compile` — `Raw/` → `Wiki/`. Groups clippings by topic, enriches existing articles instead of duplicating (checks `Wiki/index.md` first), runs a close-check for source fidelity, updates index + log, flips `compiled` flag.
- `wiki-search` — reads `Wiki/index.md` then drills into articles; returns synthesized answer with wikilink citations.
- `wiki-save` — files a conversation answer into `Wiki/` with `origin: conversation` frontmatter. Updates existing article if one matches, else creates.
- `wiki-lint` — read-only audit (contradictions, unsourced claims, overloaded articles, under-used sources, AI-voice tone). Suggests, never edits.

## Pre-loaded context convention

Every operational skill uses Claude Code's `!command` injection in its SKILL.md frontmatter/body to splice shell output (available vaults, `Wiki/index.md`, uncompiled clippings, `find` listings) in **before** Claude reads the skill body. When editing a skill, the "Pre-loaded Context" section near the top of its SKILL.md is where those commands live — changing what a skill "knows on entry" means changing those `!` commands, not adding tool calls inside the body.

## Editing the contract

To change defaults for **new** workspaces: edit `skills/wiki-init/assets/contract-template.md`. To change rules for an **existing** workspace: edit the `## Wiki` section of that workspace's `CLAUDE.md` directly — the runtime skills read whatever the local file says, not this repo.

## Distribution

Skills are installed by copying folders into `~/.claude/skills/` (user-scoped) or a target's `.claude/skills/` (project-scoped). There is no package manager or install script — `cp -R` is the install path.

---
> Source: [SantiagoBobrik/llm-wiki-kit](https://github.com/SantiagoBobrik/llm-wiki-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
