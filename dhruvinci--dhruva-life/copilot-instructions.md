## dhruva-life

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a terminal-style personal website for Dhruva Chakravarthi built with Next.js. The site mimics a command-line interface where visitors can interact with content through typed commands.

## Development Commands

**Development:**
- `pnpm dev` or `npm run dev` - Start development server
- `pnpm build` or `npm run build` - Build for production
- `pnpm start` or `npm run start` - Start production server
- `pnpm lint` or `npm run lint` - Run ESLint

**Notes:**
- ESLint and TypeScript errors are ignored during builds (see next.config.mjs)
- Uses pnpm as the primary package manager

## Architecture

**Core Concept:**
The application simulates a terminal interface where users type commands to explore content. All user interactions happen through command execution rather than traditional navigation.

**Key Components:**
- `components/terminal.tsx` - Main terminal interface with command input/output
- `hooks/use-terminal.tsx` - Terminal state management and command execution
- `lib/commands.tsx` - All available commands and their implementations
- `components/boot-screen.tsx` - Initial loading animation
- `components/command-palette.tsx` - ⌘K fuzzy search interface

**Command System:**
- Commands are defined in `lib/commands.tsx` with categories: Core, Utilities, Easter Eggs
- Case-insensitive command matching
- Command history stored in localStorage
- Alias system for custom shortcuts
- Auto-completion suggestions

**Content Structure:**
- `content/projects.ts` - Project portfolio data
- `content/logs.ts` - Weekly changelog entries  
- `content/writing.ts` - Blog posts and essays
- Static content rendered through command execution

**Styling:**
- Uses shadcn/ui components with "new-york" style
- Custom color palette: terracotta, sage, bone, olive, ochre
- Tailwind CSS with CSS variables for theming
- Monospace font for terminal aesthetic

**State Management:**
- React hooks for local state
- localStorage for persistence (history, aliases, theme)
- Custom events for inter-component communication

## Key Files

- `app/page.tsx` - Main page component with terminal integration
- `components/terminal-input.tsx` - Command input handling with history navigation
- `components/output-block.tsx` - Renders command output blocks
- `lib/animations.tsx` - ASCII art animations for easter eggs
- `components/mobile-input-bar.tsx` - Mobile-friendly input interface

## Development Notes

- The site uses Next.js App Router
- Images are unoptimized for static export compatibility
- All animations are CSS-based or React state-driven
- Terminal commands can trigger cross-component actions via custom events
- Content is statically defined but could be extended with CMS integration

---
> Source: [dhruvinci/dhruva-life](https://github.com/dhruvinci/dhruva-life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
