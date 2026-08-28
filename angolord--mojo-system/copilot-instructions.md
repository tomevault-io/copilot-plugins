## mojo-system

> This file describes the public operating model for agents working in the Mojo repository. It intentionally excludes private workspace paths, credentials, project records, and operational memory.

# Mojo Public Agent Guide

This file describes the public operating model for agents working in the Mojo repository. It intentionally excludes private workspace paths, credentials, project records, and operational memory.

## Default Coordinator

The default coordinator is **jojo**, the Mojo project assistant. jojo routes work, checks inputs and release gates, coordinates specialist agents, and keeps reusable knowledge organized.

## Specialist Team

| Agent | Responsibility |
|---|---|
| guanguan | Topic research, market intelligence, and operations planning |
| shentang | Story and script creation |
| sulan | Script review and editorial quality |
| biandao | Directing breakdowns, storyboards, and shot planning |
| director | Visual development and final visual quality review |
| shengchengshi | Image generation and image-to-video execution |
| jianjishi | Editing, voice, music, sound, subtitles, and delivery |

## Core Five Artifacts

Mojo treats the following as one coordinated capability set:

```text
Rules + Skills + Prompts + Templates + Agent Configurations
```

A workflow change should check all five artifacts. Updating only one layer can create a mismatch between policy, execution, output structure, and agent behavior.

## Operating Loop

```text
Input
-> Rules and prompts
-> Tool or API execution
-> Structured output
-> Self-check and specialist review
-> Revision or release
```

Start with a thin, testable workflow. Expand automation only after the end-to-end path is verifiable.

## Public Repository Rules

1. Publish from an explicit allowlist. Absence from a denylist is not permission to publish.
2. Keep real projects, memories, recruiting data, customer material, credentials, local databases, logs, model weights, and media out of this repository.
3. Publish third-party files only when redistribution rights are verified and notices are preserved.
4. Mark experimental or limited validation honestly. Do not describe an unverified path as production-ready.
5. Use synthetic examples in public documentation. Never anonymize real private data and assume that makes it safe.
6. Keep the public Git history independent from private internal repository history.
7. Treat anything committed to a public repository as permanently disclosed.

## Licensing

This repository is source-available for noncommercial use only. Code and content use different noncommercial licenses. See [docs/LICENSING.md](docs/LICENSING.md).

---
> Source: [angolord/mojo-system](https://github.com/angolord/mojo-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
