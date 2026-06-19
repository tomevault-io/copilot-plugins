## dotdot

> This document describes how Antigravity AI assisted in developing the DotDot game.

# Antigravity's Role in DotDot Development

This document describes how Antigravity AI assisted in developing the DotDot game.

---

## Overview

**DotDot** was developed in collaboration with Antigravity.

## What Antigravity Did

### 1. Project Initialization
- Created project structure.
- Set up `package.json` with Jest.

### 2. Game Logic Implementation
- Implemented `GameState` class in `web/js/game.js`.

### 3. UI Development
- Created SVG-based responsive rendering in `web/js/ui.js`.
- Implemented responsive grid sizing for large grids (30x30).

### Collaboration Workflow
1. **Branching Strategy**:
   - `master` is protected. Direct commits are **FORBIDDEN**.
   - All changes must go through a feature branch: `claude/<feature-name>`.
   - Feature branches must be merged via Pull Request (PR).

2. **Branch Naming**:
   - Format: `claude/<descriptive-name>`
   - Example: `claude/fix-pencil-colors`

3. **Deployment**:
   - Merging to `master` triggers deployment to `dhanjit.me`.

4. **Testing**:
   - All PRs must pass `npm test` before merging.

5. **Commits**:
   - Messages must be **fully descriptive**.
   - Use imperative mood (e.g., "Add feature" not "Added feature").

## Tech Stack
- **Frontend**: Vanilla HTML5, CSS3, JavaScript (ES6+).
- **Rendering**: SVG (Scalable Vector Graphics).
- **Testing**: Jest.
**Date**: January 2026
**AI Model**: Google Deepmind Antigravity

---
> Source: [dhanjit/dotdot](https://github.com/dhanjit/dotdot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
