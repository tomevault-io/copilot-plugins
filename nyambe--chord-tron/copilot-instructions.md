## chord-tron

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chord-Tron is a specialized music theory application built with Nuxt 4 that provides a systematic approach to diatonic chord construction and guitar chord-melody study. It uses a unique "note//skip note" construction methodology rather than traditional chord theory.

## Core Concept

The application constructs chords by taking every other note from a given scale starting at any degree, then derives various harmonic objects (shells, triads, dyads) as subsets of these base chords. This creates an "elevator study system" with progressive complexity levels (0-8) for chord-melody practice.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Generate static files
npm run generate
```

## Project Structure

This is a Nuxt 4 project with the new directory structure:
- `app/` - All application code (components, pages, composables, etc.)
- `docs/` - Comprehensive technical documentation
- `public/` - Static assets
- `nuxt.config.ts` - Nuxt configuration

## Key Dependencies

- **Nuxt 4**: Latest framework with breaking changes from v3
- **Nuxt UI 3**: UI component library with significant breaking changes from v2
- **TypeScript**: Full type safety across the application
- **ESLint**: Code linting with Nuxt's ESLint module

## Architecture Philosophy

**API-First Design**: All chord theory, scale processing, and voicing generation logic should be implemented in server API endpoints (`/api/` routes). The frontend handles only UI, visualization, and user interaction.

**Planned API Structure**:
- `/api/chords/generate` - Scale-based chord generation
- `/api/voicings/elevator` - Elevator study system levels
- `/api/analysis/identify` - Note identification and analysis

## Core Technical Concepts

### Diatonic Chord Construction
- Input: Scale + Starting Degree
- Process: Extract every other note from scale position
- Output: Base chord structure with scale positions tracked

### Elevator Study System
Progressive complexity levels:
- Level 0: Single notes
- Level 1: Thirds (dyads)
- Level 2: Triads
- Level 3: Shells (structural subsets)
- Level 4: Octaves
- Level 5-7: Drop voicings (Drop 2, Drop 3, Drop 2&4)
- Level 8: Double octaves

### Shell Extraction
Shell chords are NOT traditional 1-3-7 voicings but specific subsets of the base chord at positions [1,3,4] or [1,2,4].

## Development Notes

### Nuxt UI 3 Migration
- Significant breaking changes from v2 - check migration guide
- Script setup should be placed above template blocks
- Use Nuxt UI 3 components, not legacy versions

### Music Notation Rendering
- Plan to integrate VexChords for guitar chord diagrams
- Focus on fretboard visualization and chord chart generation
- Ensure accuracy of musical notation rendering

### Type Safety
All music theory data structures should be fully typed:
- `ScaleContext` - Scale notes and construction parameters
- `DiatonicChord` - Base chord with derived objects
- `HarmonicObject` - Notes with structural positions and inversions

## Testing Strategy

- Use Playwright for E2E testing of musical accuracy
- Test notation rendering across different screen sizes
- Validate chord diagrams match actual fretboard positions
- Ensure cross-browser compatibility for music notation

## File Structure Priorities

When creating new files:
1. API endpoints in `/server/api/` for music theory logic
2. Components in `/app/components/` for UI elements
3. Composables in `/app/composables/` for reusable logic
4. Pages in `/app/pages/` for route definitions
5. Utils in `/app/utils/` for helper functions

## Development Workflow

1. Implement music theory algorithms in server API first
2. Create TypeScript interfaces for all musical data structures
3. Build UI components that consume API data
4. Test with Playwright for musical accuracy
5. Optimize for mobile responsiveness using Tailwind CSS
6. Use Sequencial Thinking to plan tasks

## Key Constraints

- Never use traditional chord theory interval naming
- All harmonic objects derive from diatonic scale construction
- Maintain structural position tracking (not interval measurements)
- Focus on guitar fretboard application and chord-melody study

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nyambe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nyambe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
