## datacenter-planner

> - **Build**: `npm run build` (Vite production build)

# Agent Guidelines for 2D-3D Converter

## Build/Lint/Test Commands

- **Build**: `npm run build` (Vite production build)
- **Dev server**: `npm start` (Vite dev server on localhost:5173)
- **Preview**: `npm run preview` (Preview production build)
- **Lint**: `npx eslint .` (ESLint with recommended rules)
- **Test**: No test framework configured - add tests using Jest or similar

## Code Style Guidelines

### Imports & Modules

- Use ES6 import/export syntax
- Group imports: external libraries first, then local modules
- Use relative paths for local imports (`./2d/floor2d.js`)

### Naming Conventions

- **Variables/Functions**: camelCase (`createGrid`, `init3D`)
- **Constants**: UPPER_CASE (`MODELS_NAME`)
- **Files**: camelCase with descriptive names (`scene3d.js`, `pathsTo3d.js`)

### Formatting

- 2-space indentation
- No semicolons
- Single quotes for strings
- Arrow functions preferred for callbacks
- Export functions at top of file after imports

### Error Handling

- Use console.log for debugging (seen in resize handler)
- Basic error prevention (null checks before operations)

### Architecture

- Separation of concerns: 2D logic in `/2d/`, 3D logic in `/3d/`
- Event-driven architecture with DOM event listeners
- Functional programming style with pure functions where possible

### Dependencies

- Three.js for 3D rendering
- Paper.js for 2D canvas manipulation
- Bootstrap for UI components
- Vite for build tooling

---
> Source: [2listic/datacenter-planner](https://github.com/2listic/datacenter-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
