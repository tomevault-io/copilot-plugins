## portfolio-clean

> **IMPORTANT: NEVER push to GitHub automatically**

# Claude Code Guidelines

## Git & GitHub Rules

**IMPORTANT: NEVER push to GitHub automatically**
- Only commit changes when requested
- NEVER use `git push` unless explicitly asked by the user
- The user always controls when changes go to GitHub
- Only suggest commits or mention that changes are ready to be pushed
- **EXCEPTION**: User can give explicit command to commit and push (e.g. "push this to github", "commit and push")

## Build & Development Rules

**IMPORTANT: NEVER build automatically**
- NEVER use `npm run build` unless explicitly asked by the user
- NEVER use `npm run dev` unless explicitly asked by the user
- The user controls when to test, build, or run development server
- Only suggest testing when changes are ready

## Project Structure

This is a portfolio website with:
- PIXI.js slider on the home page
- Barba.js page transitions
- Scrollable project detail pages
- Responsive design with mobile optimizations

## Development Commands (USER CONTROLLED)

- `npm run dev` - Start development server (user runs this)
- `npm run build` - Build for production (user runs this)
- User tests locally before any commits

## Known Issues Fixed

- Wheel event listener was blocking scroll on project pages
- Project pages now have proper overflow: auto for mouse scroll detection
- Barba.js transitions properly manage scroll position and event listeners

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Artwalters) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
