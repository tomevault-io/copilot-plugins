## gauntlet-03-1

> **Project:** Klippy - Desktop Video Editor (Electron + React)

# Gauntlet-03 Agent System

**Project:** Klippy - Desktop Video Editor (Electron + React)

This document describes how to use the three agent system: Brenda → Pam → Caleb.

---

## Agent Workflow

```
Brenda (User Stories) → Pam (PRD) → Caleb (TODO + Implementation)
```

### Brenda — Creates User Stories

**File:** `agents/brenda-agent.md`
**Usage:** `/brenda prd-mvp.md` or `/brenda [feature-name]`

**Output:**
- User stories in "As a... I want... So that..." format
- Acceptance criteria (testable conditions)
- Dependencies between features
- Complexity: Simple/Medium/Complex
- Phase: 1, 2, 3, or 4

**Reference:** `prd-mvp.md` (8 features in 4 phases)

---

### Pam — Creates PRD

**File:** `agents/pam-agent.md`
**Usage:** `/pam [user-story]` or `/pam [feature-name]`

**Input:**
- User story from Brenda

**Output:**
- **PRD**: `prds/s[number]-[feature-name]-prd.md`
  - Preflight questionnaire (11 questions)
  - Summary, scope, UX flow
  - Functional requirements with acceptance gates
  - Data model (TypeScript)
  - IPC APIs (Electron main process)
  - Components to create/modify
  - Testing & acceptance gates
  - Definition of done
  - **Example**: `prds/s1-application-launch-prd.md`

**Reference:** `prd-template.md`, `prd-mvp.md`

---

### Caleb — Creates TODO & Implements Features

**File:** `agents/caleb-agent.md`
**Usage:** `/caleb [feature-name]`

**Process:**
1. Read user story + PRD
2. Create TODO: `todos/s[number]-[feature-name]-todo.md` (breaks down PRD into implementation tasks)
3. Get user approval on TODO before implementing
4. Create branch: `feat/{feature-slug}`
5. Implement tasks in TODO order
6. Check off each task immediately after completion
7. Verify acceptance gates pass
8. Wait for user testing
9. Commit changes (after user approval)
10. Create PR to `develop` branch

**Output:**
- **TODO**: `todos/s[number]-[feature-name]-todo.md` (implementation breakdown)
- Electron main process (in `src/main/`)
- React frontend (components in `src/components/`)
- Updated TODO with all tasks checked off
- PR to `develop` branch

---

## Quick Start

### Single Feature Workflow

```bash
# 1. Brenda creates user story from feature
/brenda [feature-name]
# Output: User story with acceptance criteria (in USER_STORIES.md)

# 2. Pam creates PRD (specify story number)
/pam s[number]-[feature-name]
# Output: prds/s[number]-[feature-name]-prd.md
# Example: /pam s1-application-launch

# 3. Caleb creates TODO and implements (references story number)
/caleb s[number]-[feature-name]
# Output: todos/s[number]-[feature-name]-todo.md + Code + PR to develop
# Example: /caleb s1-application-launch
```

### Build All Features

```bash
# 1. Brenda breaks down prd-mvp.md into user stories
/brenda prd-mvp.md
# Output: USER_STORIES.md with 8 user stories (1 per feature)

# 2. For each story, Pam creates PRD (in story order)
/pam s1-application-launch
/pam s2-video-import
/pam s3-library-view
/pam s4-timeline-view
/pam s5-trim-functionality
/pam s6-preview-player
/pam s7-export
/pam s8-session-persistence
# Output: prds/s[number]-*.md files

# 3. Parallel build with Caleb (creates TODO + implements features respecting dependencies)
/caleb s1-application-launch
/caleb s2-video-import  # depends on s1
/caleb s3-library-view  # depends on s2
/caleb s4-timeline-view # depends on s3
... (implement in dependency order, can parallelize within phase)
# Output: todos/s[number]-*.md files + Code + PRs to develop
```

---

## Key Files

### Templates

- **`agents/prd-template.md`** — PRD format with 12 sections (Preflight → Risks)
- **`agents/todo-template.md`** — TODO format with 8 sections (Pre-Implementation → PR & Merge)
- **`agents/brenda-agent.md`** — Brenda's instructions (creates user stories)
- **`agents/pam-agent.md`** — Pam's instructions (creates PRD)
- **`agents/caleb-agent.md`** — Caleb's instructions (creates TODO + implements features)

### Reference Documents

- **`prd-mvp.md`** — Product requirements: 8 features in 4 phases
  - Phase 1: App Launch + Video Import
  - Phase 2: Library View + Timeline View
  - Phase 3: Trim + Preview Player
  - Phase 4: Export + Session Persistence

### Generated Documents

**Naming Convention**: `s[story-number]-[feature-name]-prd.md` and `s[story-number]-[feature-name]-todo.md`

```
prds/
├── s1-application-launch-prd.md
├── s2-video-import-prd.md
├── s3-library-view-prd.md
├── s4-timeline-view-prd.md
├── s5-trim-functionality-prd.md
├── s6-preview-player-prd.md
├── s7-export-prd.md
└── s8-session-persistence-prd.md

todos/
├── s1-application-launch-todo.md
├── s2-video-import-todo.md
├── s3-library-view-todo.md
├── s4-timeline-view-todo.md
├── s5-trim-functionality-todo.md
├── s6-preview-player-todo.md
├── s7-export-todo.md
└── s8-session-persistence-todo.md
```

**Story Mapping**:
- s1 = Story 1: Application Launch
- s2 = Story 2: Video Import
- s3 = Story 3: Library View
- s4 = Story 4: Timeline View
- s5 = Story 5: Trim Functionality
- s6 = Story 6: Preview Player
- s7 = Story 7: Export
- s8 = Story 8: Session Persistence

---

## Tech Stack

- **Desktop Framework**: Electron (Node.js main process)
- **Frontend**: React (TypeScript)
- **Video Processing**: FFmpeg (bundled via ffmpeg-static)
- **Build**: Vite + npm
- **Platform**: macOS (primary), Windows (secondary)

---

## Development Workflow

### Branch Strategy

- **Base branch**: `develop` (never `main`)
- **Branch naming**: `feat/{feature-slug}` (e.g., `feat/video-import`)
- **PR target**: Always `develop`

### Key Points

1. **TODO is the specification** — Follow it step-by-step
2. **Check off tasks immediately** — After completing each task
3. **Test gates from PRD** — Verify happy path, edge cases, error handling
4. **Wait for user testing** — Before committing code
5. **Commit logically** — Group related changes
6. **Create PR with summary** — Link to user story + PRD

---

## 8 Features (prd-mvp.md)

| # | Feature | Phase | Status |
|----|---------|-------|--------|
| 1 | Application Launch | 1 | Pending |
| 2 | Video Import | 1 | Pending |
| 3 | Library View | 2 | Pending |
| 4 | Timeline View | 2 | Pending |
| 5 | Trim Functionality | 3 | Pending |
| 6 | Preview Player | 3 | Pending |
| 7 | Export | 4 | Pending |
| 8 | Session Persistence | 4 | Pending |

---

## Dependencies

Feature dependencies (build in this order):

1. **Phase 1** (Foundation)
   - REQ-1: Application Launch
   - REQ-2: Video Import

2. **Phase 2** (Core Editing)
   - REQ-3: Library View
   - REQ-4: Timeline View

3. **Phase 3** (Edit & Preview)
   - REQ-5: Trim Functionality
   - REQ-6: Preview Player

4. **Phase 4** (Export & Polish)
   - REQ-7: Export
   - REQ-8: Session Persistence

---

**See individual PRD files for detailed requirements, test gates, and acceptance criteria.**

---
> Source: [finessevanes/gauntlet-03.1](https://github.com/finessevanes/gauntlet-03.1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
