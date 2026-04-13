## mches-ches-study

> Next.js 14+ App Router static blog for CHES/MCHES certification exam prep. 81 YouTube video lessons across 10 sections. Static export hosted on GitHub Pages.

# CHES & MCHES Study Guide

## Project Overview

Next.js 14+ App Router static blog for CHES/MCHES certification exam prep. 81 YouTube video lessons across 10 sections. Static export hosted on GitHub Pages.

## Tech Stack

- Next.js 16 (App Router, TypeScript, `output: 'export'`)
- SCSS with BEM methodology (CSS Modules)
- gray-matter + remark/rehype for markdown parsing
- Cookie-based progress tracking (no backend)
- Husky + lint-staged + ESLint + Prettier + Stylelint

## Commands

- `npm run dev` — local development server
- `npm run build` — static export to `out/`
- `npm run lint` — run ESLint
- `npx stylelint "src/**/*.scss"` — run Stylelint
- `npx prettier --check .` — check formatting

## Architecture

- `content/course.md` — single source markdown with all 81 lessons
- `src/lib/content.ts` — parses course.md into Section[] and Lesson[] at build time
- `src/hooks/useProgress.ts` — client-side cookie progress tracking
- Routes: `/[sectionSlug]/[lessonSlug]/` for lessons

## Permissions

- Bash: allowed for all npm/npx commands, git, file operations within this project directory
- File read/write: all files within C:\MCHES-CHES-STUDY

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Angelreyes805)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Angelreyes805)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
