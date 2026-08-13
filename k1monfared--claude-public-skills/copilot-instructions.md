## claude-public-skills

> <!-- What this project does and its purpose -->

# Claude Instructions

## Project Overview
<!-- What this project does and its purpose -->

## Project Stage Definitions

### 1. Initial Setup
- Repository created, basic structure in place
- Template files copied and configured
- Ready to start development

### 2. Proof of Concept (POC)
- Validating core concept/technology
- Experimenting with approaches
- Not production-ready
- Goal: Answer "Can this work?"

### 3. Minimum Viable Product (MVP)
- First usable version with core features
- May have bugs blocking demo/production
- Focus on core functionality only
- Goal: Get something working end-to-end

### 4. Beta Release
- Feature-complete for v1.0 scope
- Functional but missing polish features
- Known list of features X, Y, Z to add
- May have active testers/users
- Goal: Refinement and feature completion

### 5. Production Ready
- Stable, tested, documented
- All planned features implemented
- Ready for real-world use
- Active maintenance

### 6. Maintenance Mode
- Core features complete and stable
- Nice-to-have features queued (A, B, C)
- Only accepting specific contributions
- Minimal active development

## Development Commands
All project commands are defined in `.claude/commands.env`

## Git Workflow
- Run tests before every commit/push
- Only push if all tests pass
- Use descriptive commit messages

## Implementation Rules
**No creative liberties** - Only implement what is explicitly discussed and decided
- Never invent UI elements, charts, or features without user approval
- Never create placeholder/demo content or hardcoded values
- Always ask "what should this show?" before creating any display elements

**Data-driven approach** - All content must come from files
- Create data files first, then build components that read from them
- For mockups: create realistic data files, never hardcode values
- Example: inventory dashboard needs `data/inventory.json`, not `const items = [...]`

**Decision-first workflow**
1. Discuss what to build (purpose, data, layout)
2. Create data structure/files
3. Build components that consume the data
4. Document everything as you go at each step

## Documentation Format
**Use loglog format** for all documentation and notes
- Create documentation as `*.log` files using loglog syntax
- Convert to markdown when needed: `loglog file.log > file.md`
- Follow loglog syntax from https://github.com/k1monfared/loglog
- Installation: `pip install loglog` or `cargo install loglog`

## Code Conventions
<!-- Coding style, naming conventions, frameworks used -->

## Testing Instructions
<!-- How to run tests, test patterns to follow -->

## Architecture Notes
<!-- Key directories, important files, design patterns -->

## Environment Setup
<!-- Dependencies, environment variables, setup steps -->

## Project Status Tracking

### STATUS.log File
- Maintain project status in `STATUS.log` file (loglog format)
- Update "Last Updated" date when making changes
- Track current stage, development mode (Claude/Manual/Hybrid), and next milestone
- Run `/home/k1/public/update_project_status.sh` after updates to sync master status file

### Status Badges
Add to top of README files for quick visual reference:

**Status**: [🔴 POC | 🟡 MVP | 🔵 Beta | 🟢 Production | ⚫ Maintenance] | **Mode**: [🤖 Claude Code | 👤 Manual | 🔀 Hybrid] | **Updated**: YYYY-MM-DD

### Converting to Markdown
Convert loglog files to markdown when needed:
```bash
loglog STATUS.log > STATUS.md
loglog readme.log > README.md
```

---
> Source: [k1monfared/claude_public_skills](https://github.com/k1monfared/claude_public_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
