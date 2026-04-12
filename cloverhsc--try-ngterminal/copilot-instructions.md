## try-ngterminal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Angular 20 application called "terminalScreen" that appears to be focused on terminal/logging functionality. The project uses:

- Angular 20 with standalone components
- TypeScript with strict mode enabled
- SCSS for styling
- Karma/Jasmine for testing
- ng-terminal and @xterm/xterm libraries for terminal functionality

## Development Commands

### Development Server
```bash
ng serve
# or
npm start
```
Starts development server at http://localhost:4200/

### Building
```bash
ng build              # Production build
ng build --watch      # Development build with watch mode
npm run build         # Production build
npm run watch         # Development build with watch
```

### Testing
```bash
ng test               # Run unit tests with Karma
npm test              # Run unit tests
```

### Code Generation
```bash
ng generate component component-name
ng generate --help    # See all available schematics
```

## Architecture

### Application Structure
- **App Component** (`src/app/app.ts`): Root component with RouterOutlet
- **Routing**: Simple routing setup redirecting root to `/log` route
- **Log Component** (`src/app/log/log.ts`): Main log component (currently empty implementation)
- **Log Service** (`src/app/log/logService/log.ts`): Injectable service for log functionality

### Key Features
- Uses Angular's standalone components (no NgModules)
- Configured with strict TypeScript settings
- SCSS styling with component-level styles
- Terminal functionality integration via ng-terminal and @xterm/xterm

### File Organization
- Components follow Angular naming conventions with `.ts`, `.html`, `.scss`, and `.spec.ts` files
- Services are organized in dedicated subdirectories
- Styles use SCSS with component-scoped styling

## Configuration Notes
- Uses Angular's new application builder (`@angular/build:application`)
- Prettier configured with Angular HTML parser
- Strict TypeScript configuration with enhanced compiler options
- Production builds have bundle size budgets (500kB initial, 1MB max)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cloverhsc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cloverhsc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
