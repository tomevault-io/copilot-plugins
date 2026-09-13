## puck-code-snippet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is an Nx monorepo with a workspace-based architecture:

- **Root**: Manages workspaces, shared configuration, and build orchestration
- **`packages/puck-code-snippet`**: React component library (npm:public)
  - Exports reusable React components
  - Built with Vite, publishes to npm
- **`apps/puck-editor`**: React application (npm:private)
  - Main editor application
  - Depends on the `puck-code-snippet` package

The TypeScript configuration uses a custom `puck-code-snippet` condition (tsconfig.base.json:19) for module resolution.

## Common Commands

### Development
```bash
# Start dev server for a specific project
npx nx dev puck-editor
npx nx dev puck-code-snippet

# Build a project (automatically builds dependencies)
npx nx build puck-editor
npx nx build puck-code-snippet

# Watch mode - rebuild dependencies on changes
npx nx watch-deps puck-editor
npx nx watch-deps puck-code-snippet
```

### Testing & Quality
```bash
# Run tests for a project (auto-builds dependencies first)
npx nx test puck-code-snippet

# Run linting
npx nx lint puck-editor
npx nx lint puck-code-snippet

# Type checking
npx nx typecheck puck-editor
npx nx typecheck puck-code-snippet
```

### Build & Preview
```bash
# Build for production
npx nx build puck-code-snippet

# Preview production build locally
npx nx preview puck-editor
```

### Nx Utilities
```bash
# List all projects
npx nx show projects

# Show project details
npx nx show project <project-name>

# Run command for all affected projects
npx nx affected -t build
npx nx affected -t test
```

## Architecture Notes

- **Build Dependencies**: Tests automatically depend on `^build` (nx.json:42-46), ensuring dependencies are built before testing
- **Parallel Execution**: Most targets support parallelism for faster builds
- **Cache**: Build, test, lint, and typecheck tasks are cached by Nx
- **Vite**: Used for both building and dev serving across all projects
- **Vitest**: Testing framework with coverage support (coverage output to `{projectRoot}/test-output/vitest/coverage`)

## Code Quality Guidelines

**Before completing any task, always:**

1. **Run linting** on modified projects:
   ```bash
   npx nx lint <project-name>
   ```

2. **Run type checking** on modified projects:
   ```bash
   npx nx typecheck <project-name>
   ```

3. **Clean up unused files** - Remove any files, imports, or dependencies that are no longer needed

4. **Verify builds pass** for affected projects:
   ```bash
   npx nx build <project-name>
   ```

---
> Source: [Burzmalian/puck-code-snippet](https://github.com/Burzmalian/puck-code-snippet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
