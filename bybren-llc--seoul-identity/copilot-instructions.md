## seoul-identity

> This file provides guidance to Gemini CLI when working with this repository.

# GEMINI.md

This file provides guidance to Gemini CLI when working with this repository.

## Project Overview

This is a screenwriting project for "Seoul Identity" by Scott Graham. The screenplay is written in Fountain format (.fountain), a plain-text markup language for screenwriting.

This repository uses a **multi-agent screenwriting harness** adapted from the WTFB Safe Agentic Workflow methodology.

## Quick Start Commands

### Workflow
```
/start-scene [description]  - Begin scene work with context
/end-session               - Complete session, commit progress
/check-format              - Validate Fountain syntax
/check-continuity          - Check consistency across script
/new-branch [type] [desc]  - Create properly named branch
```

### Writing
```
/new-scene [location]      - Add formatted scene heading
/new-character [name]      - Add character with tracking
/format-dialogue [name]    - Format dialogue block
/add-transition [type]     - Insert transition
/add-note [content]        - Add writer's note
```

### Analysis
```
/page-count                - Current page estimate
/scene-list                - List all scenes
/analyze-structure         - Structure analysis
/analyze-character [name]  - Character arc analysis
```

### Export
```
/export-pdf                - Generate industry-standard PDF
/export-fdx                - Generate Final Draft XML
/export-html               - Generate HTML preview
/export-all                - Generate all formats
```

## 11-Agent Team

| Agent | Responsibility |
|-------|---------------|
| **Story Analyst** | Beat breakdown, requirements decomposition |
| **Story Architect** | Three-act structure, narrative arc |
| **Dialogue Writer** | Character voice, subtext, polish |
| **Scene Writer** | Action lines, visual storytelling |
| **Continuity Editor** | Timeline, character, prop consistency |
| **Script Supervisor** | Quality gate, format validation |
| **Standards Reviewer** | Industry compliance |
| **Scene Annotator** | Notes, synopses, organization |
| **Research Specialist** | Location, historical, technical accuracy |
| **Production Coordinator** | Export, format conversion |
| **Session Manager** | Workflow coordination, git integration |

## Agent Profiles

@.gemini/agents/story-analyst.md
@.gemini/agents/story-architect.md
@.gemini/agents/dialogue-writer.md
@.gemini/agents/scene-writer.md
@.gemini/agents/continuity-editor.md
@.gemini/agents/script-supervisor.md
@.gemini/agents/standards-reviewer.md
@.gemini/agents/scene-annotator.md
@.gemini/agents/research-specialist.md
@.gemini/agents/production-coordinator.md
@.gemini/agents/session-manager.md

## Skills (Knowledge Base)

### Core Syntax
@.gemini/skills/fountain-syntax/SKILL.md
@.gemini/skills/scene-headings/SKILL.md
@.gemini/skills/character-dialogue/SKILL.md
@.gemini/skills/action-description/SKILL.md
@.gemini/skills/transitions/SKILL.md
@.gemini/skills/title-page/SKILL.md

### Craft
@.gemini/skills/story-structure/SKILL.md
@.gemini/skills/character-arcs/SKILL.md
@.gemini/skills/dialogue-craft/SKILL.md
@.gemini/skills/scene-analysis/SKILL.md

### Production
@.gemini/skills/format-export/SKILL.md
@.gemini/skills/page-estimation/SKILL.md
@.gemini/skills/continuity-tracking/SKILL.md

## Fountain Format Reference

### Scene Headings
```fountain
INT. COFFEE SHOP - DAY
EXT. SEOUL STREET - NIGHT
INT./EXT. CAR (MOVING) - CONTINUOUS
```

### Character & Dialogue
```fountain
SARAH
(hesitant)
I don't know about this.

JOHN (V.O.)
Trust me.
```

### Korean Dialogue (Bilingual)
```fountain
SUNG-HO
(in Korean; subtitled)
야! 이 미친 자식아.
*Ya! You crazy bastard.*
```

### Extensions
- `(V.O.)` - Voice Over
- `(O.S.)` - Off Screen
- `(CONT'D)` - Continued
- `^` after name - Dual dialogue

### Action
Present tense, visual, economical paragraphs.

### Transitions
```fountain
CUT TO:
FADE OUT.
>SMASH CUT TO:
```

### Notes & Sections
```fountain
[[This is a note - hidden in output]]
# Act One
## Sequence 1
= Synopsis text here
```

## Repository Structure

```
seoul-identity.fountain        Main screenplay

source-materials/              Original source documents
exports/                       Generated outputs (pdf/, fdx/, html/)

.gemini/
├── agents/        11 agent profiles
├── commands/      17 slash commands (.toml)
├── skills/        13 skill modules
├── README.md      Harness documentation
├── SETUP.md       Configuration guide
└── TROUBLESHOOTING.md

scripts/           Validation scripts
docs/              Documentation (WORKFLOW.md)
```

## Validation

Run validation before commits:
```bash
npm run validate        # Full validation
npm run lint:fountain   # Fountain syntax only
npm run lint:md         # Markdown lint only
npm run lint:spell      # Spell check only
```

## Git Workflow

See `docs/WORKFLOW.md` for complete documentation.

### Branch Taxonomy

| Type | Pattern | Use Case |
|------|---------|----------|
| Protected | `main` | Production-ready drafts only |
| Scene | `scene/{slug}` | Single scene work |
| Revision | `revision/{type}` | Full-script passes |
| Character | `character/{name}` | Character-focused work |
| Structure | `structure/{change}` | Reorganization |
| Fix | `fix/{issue}` | Targeted fixes |
| Export | `export/{format}` | Export preparation |

### Commit Format

```
type(scope): description

Types: scene, dialogue, action, structure, revision, notes, format, docs, chore
```

### Definition of Done

All branches require:
- `/check-format` passes
- `/check-continuity` passes
- Spell check passes
- Self-review completed

## Operating Procedures

- **Branch Protection**: Do not push directly to main
- **Git Branch Reminder**: Create feature branch for new work
- **Commit Message Format**: Follow `type(scope): description`
- **Fountain Format**: Validate before committing
- **Post-Edit Validation**: Run `/check-format` after edits

## Key Principles

1. **Pattern Discovery First**: Check existing patterns before creating new approaches
2. **Continuity Tracking**: Maintain character, timeline, and prop consistency
3. **Format Validation**: Run `/check-format` before export
4. **Git Integration**: Commit at logical checkpoints
5. **Quality Gates**: Script Supervisor approval before delivery

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bybren-llc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
