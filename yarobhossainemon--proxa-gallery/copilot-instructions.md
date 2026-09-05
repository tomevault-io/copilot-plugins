## proxa-gallery

> Proxa Gallery is a modern Android gallery application.

# Proxa Gallery - AI Development Guide

## Project Overview

Proxa Gallery is a modern Android gallery application.

The goal is to build an offline-first, AI-powered gallery capable of searching photos using natural language.

Example searches:

- Show my cat
- Photos from Cox's Bazar
- Me wearing a blue shirt
- Screenshots
- Documents
- Sunset photos

No cloud processing should be required.

---

## Tech Stack

Language:
- Kotlin

UI:
- Jetpack Compose
- Material 3

Architecture:
- MVVM

Minimum SDK:
- 29

Target SDK:
- Latest Stable

---

## AI Rules

Never replace working code unless necessary.

Modify only files required for the requested task.

Explain every change before writing code.

Avoid deprecated Android APIs.

Prefer official AndroidX libraries.

Do not add dependencies unless requested.

---

## Code Style

Keep functions short.

Keep files reasonably small.

Avoid duplicated logic.

Use meaningful names.

Follow Kotlin conventions.

---

## Architecture

Presentation
↓
ViewModel
↓
Repository
↓
MediaStore / Room / AI

The UI must never directly read MediaStore.

---

## UI Rules

Use Material 3.

Use Jetpack Compose only.

Support dark mode.

Keep UI responsive.

Avoid hardcoded colors.

---

## Git Workflow

After every working feature:

Run the app.

Verify it works.

Commit changes.

Never commit broken code.

---

## Development Workflow

Always implement one feature at a time.

Do not generate unrelated improvements.

Preserve existing project structure.

Ask before making breaking architectural changes.

Always explain why a change is necessary.

---
> Source: [yarobhossainemon/proxa-gallery](https://github.com/yarobhossainemon/proxa-gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
