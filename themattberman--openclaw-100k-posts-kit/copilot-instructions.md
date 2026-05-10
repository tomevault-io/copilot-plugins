## openclaw-100k-posts-kit

> This repo is packaged as `100k-posts-kit`.

# AGENTS

This repo is packaged as `100k-posts-kit`.

## Purpose

Use this package to demo or run an OpenClaw-native content pipeline:

`Virlo -> Content DNA Batch -> Pattern Report -> The Forge -> Monday Drop`

Optional research layer:

`Content Radar` adds Reddit and X discussion signal when available.

## Default Agent Rules

- Read `docs/workspace-contract.md` before changing shared workspace behavior.
- Treat `~/clawd/workspace/brand/voice-profile.md` as the source of truth for production voice.
- Treat `skills/the-forge/demo-voice-profile.md` as demo-only fallback behavior.
- Prefer the local shell scripts over ad hoc manual steps when demonstrating the workflow.
- Be explicit about what is fully automated versus agent-assisted.

## Voice Handling

- Production mode: use `/voice-lab` to build the user's real voice profile.
- Demo mode: if the user wants a fast example, run with Yames first.
- Do not present Yames as the intended long-term voice.

## Core Paths

- `skills/` contains the OpenClaw skill definitions.
- `scripts/monday-drop.sh` is the main local orchestration entrypoint.
- `docs/workspace-contract.md` defines cross-skill file ownership and output locations.
- `DEMO-GUIDE.md` is the operator-facing walkthrough.
- `SETUP.md` is the public install and configuration guide.

## Release Posture

- Keep public docs free of private-project references.
- Keep required dependencies honest.
- Preserve the demo path, but label it clearly as demo behavior.

---
> Source: [TheMattBerman/openclaw-100k-posts-kit](https://github.com/TheMattBerman/openclaw-100k-posts-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
