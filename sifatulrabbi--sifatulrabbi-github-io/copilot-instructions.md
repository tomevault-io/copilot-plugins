## sifatulrabbi-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website built as an interactive terminal-first interface. The portfolio simulates a Unix-like terminal where visitors can navigate through experiences, projects, and blog content using familiar command-line commands (ls, cd, cat, etc.).

**Tech Stack:**

- React 18 with TypeScript
- Vite (development server runs on port 3000)
- React Router for routing
- TailwindCSS for styling
- Bun as the test runner (Jest)

## Common Commands

### Development

```bash
npm run dev          # Start dev server on port 3000 (DO NOT run automatically)
npm run build        # TypeScript compilation + Vite build
npm run preview      # Preview production build
npm run lint         # Run ESLint with TypeScript
npm run format-code  # Format code with Prettier
npm run test         # Run tests with Bun/Jest
```

### Testing

- Tests are located in `/tests` directory
- Use `bunx jest` to run tests (or `npm run test`)
- Test files use `.test.ts` or `.test.tsx` extensions
- Tests use Jest with Babel for TypeScript compilation

## Code Architecture

### Module-Based Structure

The codebase follows a module-based architecture where each major feature is self-contained:

**Modules** (`src/modules/`):

- `TerminalMode/` - Core terminal interface and command system
- `Blogs/` - Blog browsing and reading functionality
- `LiveApps/` - Interactive utilities (HTML viewer, markdown converter)

Each module typically contains:

- Component files
- Context provider and context definition
- Route configuration
- Module-specific services and data

### Terminal Command System

The terminal interface (`TerminalMode` module) implements a class-based command pattern where each command is a separate class implementing the `TerminalHistory` interface:

**Key classes in `terminalService.ts`:**

- `ls` - List directory contents
- `cd` - Change directory (with callback to update context)
- `cat` - Display file contents (with fuzzy matching)
- `ListExp` - Display experiences (args: none, `all`, or specific ID)
- `ListProjects` - Display projects (args: none, `all`, or specific ID)
- `PrintExperiences` - Rich HTML display of work experience
- `PrintProjects` - Rich HTML display of projects
- `PrintSkills` - Display technical skills grouped by category
- `PrintContact` - Display contact information
- `Help` - Show available commands
- `Echo`, `Time`, `Exit` - Utility commands
- `InvalidCommand` - Handles unknown commands

**Virtual File System:**

- Implemented in `directories.ts` using the `Entry` class
- Creates a tree structure of directories and files
- Files contain markdown content (experiences, projects, contact info)
- The `FileEntry` interface defines the contract for navigation
- File/directory names are normalized (lowercase, spaces removed) for fuzzy matching
- Navigation supports partial matching (e.g., "cd ex" matches "Experiences")

### Context Pattern

The project uses React Context extensively for state management:

**Terminal Context** (`terminalModeContext.tsx`):

```typescript
{
  history: TerminalHistory[]     // Command execution history
  executing: boolean             // Command in progress
  currentDir: FileEntry          // Current directory in virtual FS
  runCommand: () => Promise<void>
}
```

**Blogs Context** (`blogsContext.tsx`):

```typescript
{
  allArticles: ArticleEntry[]
  selectedArticle: ArticleItem | null
  searchQuery: string
  busy: boolean
  // ... methods for fetching and managing blog data
}
```

### Path Aliasing

TypeScript paths are configured with `@/*` alias pointing to `src/`:

```typescript
import { Component } from "@/components";
import { useTerminalMode } from "@/modules/TerminalMode/terminalModeContext";
```

### Styling Approach

- TailwindCSS with custom terminal-themed color palette (configured in `tailwind.config.cjs`)
- Terminal theme colors include:
  - Backgrounds: `terminal-background`, `terminal-surface`, `terminal-surface-hover`
  - Borders: `terminal-border`, `terminal-border-dim`
  - Text: `terminal-primary`, `terminal-secondary`, `terminal-accent`
  - Syntax: `terminal-keyword`, `terminal-string`, `terminal-function`, etc.
  - Status: `terminal-success`, `terminal-error`, `terminal-warning`, `terminal-info`
- Typography plugin enabled for markdown content rendering
- Custom fonts: Montserrat/Roboto for display, Fira Code for monospace
- Custom blink animation for cursor effect

### Markdown Rendering

- Uses `marked` library to convert markdown to HTML
- Command outputs (especially from `ListExp`, `ListProjects`, `cat`) return markdown
- Rendered HTML is displayed in the terminal interface
- DOMPurify is available for sanitizing HTML content
- Additional markdown tools: `showdown` for conversion, `turndown` for HTML to markdown

## Key Files and Architecture

- `src/Router.tsx` - Main routing configuration (Terminal, Blogs, LiveApps)
- `src/App.tsx` - Root application component with router outlet
- `src/main.tsx` - Application entry point
- `src/modules/TerminalMode/`
  - `terminalService.ts` - All terminal command class implementations
  - `directories.ts` - Virtual file system implementation with `Entry` class
  - `terminalModeContext.tsx` - Context definition for terminal state
  - `TerminalModeProvider.tsx` - Context provider with command execution logic
  - `experienceData.ts` - Experience content data
  - `projectsData.ts` - Project content data
- `src/modules/Blogs/`
  - `blogsContext.tsx` - Context definition for blog state
  - `BlogsProvider.tsx` - Context provider with API integration
  - `BrowseBlogsPage.tsx`, `ReadBlogsPage.tsx` - Blog UI pages
- `src/modules/LiveApps/` - Standalone utility apps (HTML viewer, markdown converter)
- `src/types/` - TypeScript type definitions (`terminalMode.d.ts`, `blogs.d.ts`, `index.d.ts`)
- `src/assets/data/` - Structured data for experiences and projects
- `tailwind.config.cjs` - Terminal theme configuration
- `vite.config.ts` - Vite configuration with path aliases

## Build Configuration

- **Vite**: Uses SWC plugin (`@vitejs/plugin-react-swc`) for fast React refresh, dev server on port 3000
- **TypeScript**: Strict mode enabled with ES2020 target, experimental decorators enabled
- **ESLint**: Configured for React + TypeScript, `@typescript-eslint/no-explicit-any` disabled
- **PostCSS**: Configured with Tailwind and Autoprefixer
- **Jest**: Configured with Babel presets for TypeScript and ES6+ support

## Adding New Terminal Commands

When adding new terminal commands to the TerminalMode module:

1. Create a new class in `terminalService.ts` implementing the `TerminalHistory` interface
2. The class must have:
   - `cmd: string` - The command string
   - `pwd: string` - Current working directory
   - `output: string` - Command output (can be markdown or HTML)
   - `exitCode: TerminalExitCode` (0 for success, 1 for error)
   - `exec(): TerminalExitCode` - Method that executes the command logic
3. Parse arguments from `cmd` by splitting on spaces
4. Return markdown or HTML from the `output` getter (markdown will be converted using `marked`)
5. Update the command parser in `TerminalModeProvider.tsx` to instantiate your new command class
6. Add the command to the help text in `terminalService.ts:564`

## Important Notes

- Use double quotes for strings (per user's global CLAUDE.md)
- Git operations (add, commit) should be done by the user
- Never auto-run dev server
- Testing must be done by the user - do not attempt to test with bash scripts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sifatulrabbi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
