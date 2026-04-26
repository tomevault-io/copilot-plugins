## 40k-planner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Warhammer 40k Tournament Planner - A web application for planning tournament strategies with army importing, mission deployment viewing, and flashcard study tools.

**Tech Stack:**
- Next.js 16.1.1 (App Router)
- React 19.2.3
- TypeScript 5
- Tailwind CSS 4

## Development Commands

```bash
npm run dev    # Start development server at http://localhost:3000
npm start      # Run production server
npm run lint   # Run ESLint
```

**IMPORTANT**:
- Do NOT run `npm run build` during development. The build process is slow and unnecessary for development work.
- Do NOT run `npm run dev` - the user already runs this in a separate terminal window.
- Commit after every significant change, but do NOT push unless explicitly asked.

## Architecture

### App Structure
- **App Router**: Uses Next.js App Router with client-side components
- **Main Page** (`app/page.tsx`): Tabbed interface with URL-based navigation via query params (`?tab=army|deployment|flashcards`)
- **Components** (`app/components/`): Three main feature components
- **Styling** (`app/globals.css`): Tailwind v4 with inline theme configuration

### Component Architecture

**ArmyImporter** (`app/components/ArmyImporter.tsx`)
- Text-based army list parser
- Splits input by newlines to create unit entries
- Simple import/clear functionality

**DeploymentPlanner** (`app/components/DeploymentPlanner.tsx`)
- Displays 5 mission terrain layouts (Terraform, Purge the Foe, Hidden Supplies, Linchpin, Take and Hold)
- Dropdown selector to switch between rounds
- Uses Next.js Image component with images from `/public/`

**FlashcardMaker** (`app/components/FlashcardMaker.tsx`)
- Two modes: Edit and Study
- Edit mode: Create flashcards with front/back text
- Study mode: Navigate through cards with show/hide answer
- State managed entirely in component (not persisted)

### Design System

**Theme**: Dark Thousand Sons aesthetic with blue-black and gold accents

**Colors** (defined in `app/globals.css`):
- Background: `#0a0a14` (very dark blue-black)
- Foreground: `#e0e0e0` (light gray text)
- Accent: `#C5A33E` (gold - primary)
- Dark Accent: `#4a3a0f` (dark gold)
- Border: `#1a1a2e` (dark blue borders)

## Key Patterns

- **Client Components**: All interactive components use `'use client'` directive at the top
- **URL State**: Tab state persists via query params using `useSearchParams` and `useRouter` from `next/navigation`
- **Static Assets**: Mission layout PNGs are duplicated in both `/layouts/` (source) and `/public/` (served)
- **Image Loading**: Use Next.js `Image` component with absolute paths (e.g., `/round1_terraform.png`)

## Development Guidelines

### Working with JSON Data

When inspecting or understanding the structure of JSON files (especially army list exports), **always use `jq`** to examine the structure:

```bash
# View overall structure
jq '.' /path/to/file.json

# Inspect specific paths
jq '.roster.forces[0].selections[]' /path/to/file.json

# Filter by type
jq '.roster.forces[0].selections[] | select(.type == "unit")' /path/to/file.json
```

This helps understand the data hierarchy before writing parsing code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaashif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
