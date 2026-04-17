## mdimension

> - Always complete all tasks fully - no simplifying approaches or skipping steps

# MDimension Project Rules

## Critical Execution Protocol

### Task Completion Standards
- Always complete all tasks fully - no simplifying approaches or skipping steps
- Keep tests up to date with 100% test coverage
- All tests must pass before task is complete
- Fix bugs in code, never modify tests just to make them pass
- Never run Vitest in watch mode; use `npm test` for automation

### After Implementing New Functionality
1. Create comprehensive tests:
   - Unit tests for logic and components (Vitest)
   - Integration tests for game flow
   - Playwright tests for frontend functionality
2. All tests must be in `src/tests/` or `scripts/playwright/`
3. Run ALL tests before considering task complete

## Code Style & Architecture

### Mandatory Style Guide
Follow `docs/meta/styleguide.md` without exception.

### Shader Requirements
All shaders MUST use WebGL2/GLSL ES 3.00 syntax:
- Use `in`/`out` declarations
- Use `layout` qualifiers
- Never use deprecated `attribute`/`varying`/`gl_FragColor`

## Test Memory Management

### Configuration Safeguards (DO NOT MODIFY)
- `maxWorkers: 4` in `vitest.config.ts` - prevents excessive process spawning
- `pool: 'threads'` - uses memory-efficient threading
- `environment: 'happy-dom'` - fast DOM implementation

### Writing Memory-Safe Tests
**DO:**
- Process data in batches of 100, clear arrays between batches
- Use component tests (`.test.tsx`) only for UI components
- Call `cleanup()` from @testing-library/react in test teardown

**DON'T:**
- Generate 1000+ data points in a single test without batching
- Rely on DOM for pure logic tests if not needed
- Forget to cleanup timers/listeners in afterEach

### Emergency Response
If system becomes unresponsive during tests:
```bash
killall -9 node
node scripts/cleanup-vitest.mjs
```

## Folder Usage Rules

| Activity | Required Directory |
| --- | --- |
| Browser automation (Playwright/Puppeteer) | `scripts/playwright/` |
| Physics, RNG, analytics utilities | `scripts/tools/` |
| Visual artifacts (screenshots, videos, GIFs) | `screenshots/` |
| Documentation, research notes | `docs/` |
| Temporary experiments / sandboxes | `src/dev-tools/` |
| 🚫 **FORBIDDEN** | Project root (keep pristine) |

## Tech Stack Reference

### Core Framework
- React 19.2.3 - UI library
- TypeScript 5.6.3 - Type-safe JavaScript
- Vite 7.2.7 - Build tool and dev server

### 3D Graphics & Rendering
- Three.js 0.181.0 - WebGL 3D library
- @react-three/fiber 9.4.2 - React renderer for Three.js
- @react-three/drei 10.7.7 - Three.js utilities
- @react-three/postprocessing 3.0.4 - Post-processing effects
- postprocessing 6.38.0 - Post-processing library

### UI & Styling
- Tailwind CSS 4.1.18 - Utility-first CSS framework
- Motion 12.23.26 - Animation library

### State Management
- Zustand 5.0.2 - State management

### Testing
- vitest 4.0.15 - Unit testing
- happy-dom 15.11.7 - DOM implementation for testing
- playwright 1.57.0 - E2E testing

## Mandatory Document Reads

Before working on this project, review:
- `docs/architecture.md` - Project architecture and folder structure
- `docs/testing.md` - Testing setup and development environment
- `docs/frontend.md` - Frontend setup
- `docs/meta/styleguide.md` - Code style guide (MANDATORY)
- `docs/research/nd-dimensional-react-threejs-guide.md` - Math for object creation/transformation/projection
- `docs/prd/ndimensional-visualizer.md` - Product requirements
- `docs/prd/enhanced-visuals-rendering-pipeline.md` - Rendering pipeline PRD

## Task Management

Use the todo system for any task with 3+ steps to:
- Track progress
- Maintain focus
- Ensure systematic approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michael-haufschild-gib) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
