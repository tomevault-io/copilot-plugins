## amane-kazami

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an interactive fiction project written in Ink (inkle's narrative scripting language). The story follows Amane Kazami (风见弥), a high school student experiencing severe bullying who encounters a supernatural entity.

The project is written primarily in Chinese with some English chapter titles.

## File Structure

### Main Entry Points
- `amane kazami.ink` - Root file in the project directory (contains menu structure)
- `chapters/amane kazami.ink` - Main chapter orchestrator that includes all chapter files

### Chapter Organization
The story is divided into separate chapter files in the `chapters/` directory:
- `chapter1.ink` through `chapter9.ink` - Individual story chapters
- Each chapter is included via `INCLUDE` statements in `chapters/amane kazami.ink`

### Assets
- `images/` - Contains PNG images referenced in the story using `# IMAGE:filename.png` syntax
  - Examples: `classroom.png`, `corridor.png`, `apartment_door.png`, `street.png`, etc.
- `backup/` - Contains backup versions of the main ink file

## Ink Syntax Patterns Used

### Navigation Structure
- Menu choices: `+ [choice text] -> destination_knot`
- Story progression: `* [choice text] -> destination_knot` (sticky choices that disappear after selection)
- Knots: `=== knot_name ===`
- Diverts: `-> knot_name`

### Image Integration
Images are embedded using the comment syntax:
```
# IMAGE:filename.png
```

### Text Continuation
- `<>` is used for gluing text together without line breaks

### Story Branches
The story includes multiple endings (e.g., `badend1`) based on player choices.

## Development Workflow

### Working with Chapters
When editing the story:
1. Individual chapters are in `chapters/chapter*.ink`
2. The main orchestrator is `chapters/amane kazami.ink`
3. Changes to chapter structure require updating the INCLUDE statements

### Testing Changes
This project uses Ink, which typically requires:
- Ink compiler (inklecate) to compile `.ink` files to `.json`
- An Ink player/runtime to test the interactive story
- Check for syntax errors using the Ink compiler

### Recent Development Focus
Based on git history:
- Menu interface improvements
- Chapter division and organization
- Fixing navigation issues (dot leading to jumps)
- Output file cleanup

## Story Structure Notes

### Main Menu Flow
```
main -> [开始游戏/关于本作/制作者信息]
start -> Chapter1 -> subsequent chapters
```

### Chapter Progression
Chapters follow a narrative arc with branching paths and multiple endings. The story explores themes of bullying, supernatural encounters, loneliness, and the consequences of seeking help from otherworldly entities.

### Character Names
- 风见弥 (Mi/Amane Kazami) - The protagonist
- The story involves a supernatural entity that appears to the protagonist

## Important Conventions

1. **Language**: Story content is in Chinese; maintain this when editing narrative text
2. **Image References**: Always use relative paths from the root directory (e.g., `images/filename.png`)
3. **Chapter Naming**: Follow the pattern `Chapter# - English Title` for chapter headers
4. **Knot Naming**: Use lowercase with underscores for knot names (e.g., `no_more_running`, `badend1`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kalpalevels)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kalpalevels)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
