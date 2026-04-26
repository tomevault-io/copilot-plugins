## promptmaster

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

PromptMaster is a curated collection of high-quality LLM prompts organized by category. This is a documentation-focused repository with no build system or traditional codebase.

## Repository Structure

```
prompts/           # Main content directory with categorized prompts
├── analytical/    # Policy and debate analysis prompts
├── brainstorming/ # Creative thinking and ideation
├── learning/      # Skill acquisition and education  
├── productivity/  # Meeting optimization, facilitation
├── strategic/     # Decision-making and strategic planning
├── technical/     # Code review and programming assistance
└── writing/       # Executive communication and email response

templates/         # Contains prompt-template.md for new prompts
tags/             # Tag-based navigation with tag-index.md
```

## Common Tasks

### Adding a New Prompt
1. Use the template at `templates/prompt-template.md`
2. Place in appropriate category under `prompts/`
3. Follow the standardized format: Purpose → Template → Example → Tips → Variants
4. Update `tags/tag-index.md` with relevant tags
5. Avoid em dashes (—) in all content as they signal AI-generated text

### Git Workflow
- No build or test commands needed
- Standard git commands for version control
- Commit messages should describe the prompt changes clearly

## Important Guidelines

### Content Standards
- All prompts must include concrete examples and usage tips
- Each prompt requires appropriate tags for discovery
- Maintain consistent markdown formatting across all prompts
- Focus on professional, business, and technical use cases

### Tag System
- The repository uses 73+ unique tags for categorization
- Tags are maintained in `tags/tag-index.md`
- New prompts should use existing tags where possible

## Architecture Notes

This is a content repository, not a software project. There are:
- No dependencies to install
- No build processes to run
- No tests to execute
- No linting configurations

The value is in the curated prompt content and its organization for easy discovery and use.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lengfelj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
