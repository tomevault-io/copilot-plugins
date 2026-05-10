## resume-tailor-plugin

> This repository is a dual-format Claude Code and Codex plugin. Keep both plugin manifests in sync when changing plugin metadata:

# AGENTS.md

This repository is a dual-format Claude Code and Codex plugin. Keep both plugin manifests in sync when changing plugin metadata:

- `.claude-plugin/plugin.json` for Claude Code.
- `.codex-plugin/plugin.json` for Codex marketplace packaging.
- `.agents/plugins/marketplace.json` for the Codex self-hosted marketplace index.

## Project Overview

The plugin provides one skill: `resume-tailor`. It tailors resumes to specific job descriptions with ATS optimization. There is no build system, test suite, or package manager; the project is prompt-based markdown plus plugin metadata.

## Repo Layout

- `skills/resume-tailor/SKILL.md` is the orchestrator for the multi-stage workflow.
- `skills/resume-tailor/references/` contains ATS rules, locale guidance, and section templates.
- `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` are distribution manifests.

## Verification

Validate JSON manifests after metadata changes. For skill changes, read `SKILL.md` with the referenced locale and ATS files to make sure the workflow remains consistent.

---
> Source: [olegvg/resume-tailor-plugin](https://github.com/olegvg/resume-tailor-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
