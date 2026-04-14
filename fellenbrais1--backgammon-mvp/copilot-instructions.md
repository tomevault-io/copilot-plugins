## backgammon-mvp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Commands
- Run local server: In VS Code, right-click on index.html and select 'Open with Live Server'
- Lint code: `npx eslint scripts/*.js`
- No formal test setup available (use browser testing)

## Code Style Guidelines
- Use 'use strict' at the top of JavaScript files
- Single quotes for strings, always use semicolons
- camelCase for variables/functions, clear descriptive names
- Use ESM imports/exports for modularity 
- Consistent 2-space indentation
- Group code by: IMPORTS, VARIABLES, EVENT LISTENERS, FUNCTIONS
- Use try/catch for error handling
- Use async/await for asynchronous operations
- Add descriptive comments for functions and complex logic
- Use console.logging for debugging
- Backgammon-specific: Points 1-24, Bar = 25/26, Board = 0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fellenbrais1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fellenbrais1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
