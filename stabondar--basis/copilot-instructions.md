## basis

> - `npm run dev` - Start development server (port 4321)

# Project Guidelines for stabondar-template

## Build Commands
- `npm run dev` - Start development server (port 4321)
- `npm run build` - Build production bundle
- `npm run preview` - Preview production build (port 8080)
- `npm run clean` - Remove dist folder
- `npm run lint:fix` - Run ESLint with auto-fix
- `npm run deploy` - Deploy to Vercel

## Code Style Guidelines
- **Module System**: ES modules with import/export
- **Naming**: 
  - PascalCase for classes (Accordion, EventEmitter)
  - camelCase for methods and variables
- **Formatting**: 4-space indentation, braces on new lines
- **Import Organization**:
  - External imports first, then internal imports with newlines between groups
  - Use path aliases (@utils, @modules, @transitions, etc.)
- **Architecture**:
  - Class-based with singleton pattern where appropriate
  - Event-driven using EventEmitter for communication
  - GSAP for animations (from local gsap-bonus.tgz)
  - Barba.js for page transitions
- **Error Handling**: Console logs with early returns

## Project Structure
- `src/modules` - Reusable UI components
- `src/transitions` - Page transition effects
- `src/utils` - Utility classes and helpers
- `src/pages` - Page-specific components
- `src/css` - SCSS styles with modular organization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stabondar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
