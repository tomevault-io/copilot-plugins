## voicify

> - **PREPARE commit commands but DO NOT execute them automatically**

# Git Workflow Rules

## Commit Workflow
- **PREPARE commit commands but DO NOT execute them automatically**
- When user asks to commit changes, prepare the commit workflow but let user execute commands manually
- Do NOT repeatedly ask about commits in the same session unless user explicitly requests it
- Follow the established commit workflow from [commit.md](mdc:.windsurf/workflows/commit.md):
  1. Check what was changed in all uncommitted files
  2. Split git add/commit into separate commits for connected changes if necessary
  3. Make comprehensive, concise summary of most important changes from project perspective
  4. Prepare the git add and git commit commands with conventional commits format
  5. Present the prepared commands to user for manual execution
  6. DO NOT execute git commands automatically - only prepare them

## Commit Message Format
- Use conventional commits format: `type(scope): description`
- Common types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Keep commit messages **SHORT and CONCISE**
- Use bullet points for multiple changes, but keep each point brief
- Avoid verbose descriptions - focus on essential changes only
- Reference issue numbers if applicable

## Branch Management
- Work on feature branches, not directly on main/develop
- Keep commits atomic and focused on single functionality
- Squash related commits when appropriate before merging

## Code Review Preparation
- Ensure all changes are properly tested before committing
- Include relevant documentation updates in the same commit
- Check that commit doesn't break existing functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dooshek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dooshek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
