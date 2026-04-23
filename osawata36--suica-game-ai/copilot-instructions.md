## suica-game-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a "suica-game-ai" project - an AI-developed game. The repository is currently in its initial state with only a basic README file.

## Current State

The repository contains:
- README.md with basic project description
- game.js - Main game implementation with enhanced fruit visuals
- index.html - Game HTML interface
- No build system or package configuration files yet

## Development Setup

Since this is a new project with no existing build system:
- No package.json, requirements.txt, or other dependency files exist yet
- No build, test, or lint commands are configured
- The project structure and technology stack are yet to be determined

## Git Workflow & Commit Guidelines

### Conventional Commits
All commits must follow the Conventional Commits specification:

**Format**: `<type>(<scope>): <description>`

**Types**:
- `feat`: New features
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code formatting (no functional changes)
- `refactor`: Code refactoring
- `test`: Adding/modifying tests
- `chore`: Other changes (build process, auxiliary tools, etc.)

**Commit Process**:
1. Use TodoWrite tool to plan and track tasks
2. Make changes with appropriate tools (Edit, MultiEdit, etc.)
3. Run parallel git commands to check status and changes:
   - `git status` - check working tree
   - `git diff` - view changes
   - `git log --oneline -3` - check recent commit style
4. Stage changes with `git add`
5. Commit with detailed conventional format using heredoc:
   ```bash
   git commit -m "$(cat <<'EOF'
   feat: description of the feature
   
   - Detailed bullet points of changes
   - What was added/modified
   - Why the change was made
   
   🤖 Generated with [Claude Code](https://claude.ai/code)
   
   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```
6. Verify commit success with `git status`

### Development Workflow
1. **Planning**: Use TodoWrite to break down tasks
2. **Implementation**: Make focused, single-purpose changes
3. **Commit**: One logical change per commit
4. **Update todos**: Mark tasks as completed immediately after finishing

## Technology Stack

Current implementation uses:
- HTML5 Canvas for rendering
- Vanilla JavaScript for game logic
- CSS for styling
- No external dependencies or build system

## Next Steps for Development

When adding code to this repository:
1. Determine the appropriate technology stack based on the game requirements
2. Set up the appropriate build system (package.json for Node.js, requirements.txt for Python, etc.)
3. Establish project structure and architecture
4. Configure linting and testing tools as appropriate for the chosen stack

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/osawata36) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
