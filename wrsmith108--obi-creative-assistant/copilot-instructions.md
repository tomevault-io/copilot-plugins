## obi-creative-assistant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm install` - Install all dependencies
- `npm run dev` - Start development server on port 8080 with hot reload
- `npm run build` - Create production build
- `npm run build:dev` - Create development build with source maps
- `npm run preview` - Preview production build locally
- `npm run lint` - Run ESLint to check code quality

## Architecture Overview

### Tech Stack
- **React 18.3** with TypeScript for the frontend
- **Vite** as the build tool with SWC for fast compilation
- **Adobe React Spectrum** + **shadcn/ui** for UI components
- **TanStack Query** for server state management
- **Tailwind CSS** for styling with custom design tokens
- **React Router v6** for client-side routing
- **@11labs/react** for voice capabilities

### Project Structure
- `/src/components/` - All React components, including custom app components and shadcn/ui library
- `/src/pages/` - Route-based page components
- `/src/hooks/` - Custom React hooks (keyboard shortcuts, mobile detection)
- `/src/lib/` - Utilities and simulated API layer (firefly-simulation.ts)
- Path alias: `@/` maps to `/src/` directory

### Key Architectural Patterns

#### Component Hierarchy
```
App.tsx (providers setup)
└── Index.tsx (landing page)
    └── ObiModal (main application)
        ├── ConversationInterface (chat UI)
        │   └── VoiceControls
        ├── WorkflowVisualization
        └── BrandContextManager (brand assets)
```

#### State Management
- Local state with React hooks (no global state library)
- React Query for future API integration (currently using simulated data)
- All API responses are mocked in `firefly-simulation.ts`

#### Persona System
The app supports 4 user personas, each with:
- Custom quick actions in ConversationInterface
- Tailored welcome messages
- Specific workflow suggestions
- Focused expertise areas

Personas: Creative Director, Campaign Manager, Designer, Developer

### Important Implementation Details

#### UI/UX Patterns
- macOS-style window with custom menu bar
- Dark/light theme support via next-themes
- Glass morphism effects with Tailwind utilities
- Command palette (Cmd+K) for quick actions
- Keyboard shortcuts managed by useKeyboardShortcuts hook

#### TypeScript Configuration
- Relaxed TypeScript settings (no strict null checks, implicit any allowed)
- Path aliasing configured for `@/` imports
- Separate configs for app and node environments

#### Styling Approach
- Tailwind CSS with extended theme configuration
- Custom color palette: creative-primary, creative-secondary, etc.
- Consistent spacing tokens and animations
- CSS variables for theme switching

#### Current Limitations
- No real API integration (all mocked)
- No authentication system
- No error boundaries
- No test suite configured
- State management relies on prop drilling for complex interactions

### Development Tips

1. When adding new features, follow the existing component patterns in `/src/components/`
2. Use the persona system to tailor experiences - check `personaDescriptions` in relevant components
3. All Adobe Firefly API responses are simulated in `/src/lib/firefly-simulation.ts`
4. For UI components, prefer Adobe React Spectrum components, fallback to shadcn/ui
5. Maintain TypeScript interfaces for all component props and data structures
6. Use the established color tokens and spacing values from Tailwind config

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wrsmith108)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wrsmith108)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
