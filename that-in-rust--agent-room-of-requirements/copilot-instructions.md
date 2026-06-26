## agent-room-of-requirements

> This file provides guidance to Claude Code when working in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## What This Repo Is

Private research knowledge base (agent-room-of-requirements). Contains analysis docs, thesis documents, and research notes for open source contributions.

## Git Rules — MANDATORY

### NEVER push without explicit permission

- Do NOT run `git push` unless the user explicitly says "push"
- Always ask before pushing

### NEVER reference external issue numbers in commit messages

- This repo's commits show up as cross-references on GitHub issues
- Commit messages must NEVER contain patterns like `org/repo#123`, `apache/iggy#2739`, or bare `#123`
- Strip all issue number references from commit messages
- Put issue references inside file content only, never in commit messages
- BAD: `Add deep analysis for MongoDB connector (apache/iggy#2739)`
- GOOD: `Add MongoDB connector deep analysis for iggy`

---
> Source: [that-in-rust/agent-room-of-requirements](https://github.com/that-in-rust/agent-room-of-requirements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
