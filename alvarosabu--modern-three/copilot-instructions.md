## modern-three

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Modern ThreeJS is a functional 3D graphics boilerplate using Three.js with Vite. It serves as a starting point for 3D web projects, featuring shader support, debug UI, and modern TypeScript configuration.

## Commands

```bash
pnpm dev          # Development server (localhost:3000)
pnpm build        # TypeScript check + Vite build
pnpm lint         # Run ESLint
pnpm lint:fix     # Auto-fix linting issues
pnpm preview      # Preview production build
```

## Architecture

### Core Systems (src/core/)

- **renderer.ts**: Creates WebGLRenderer, Scene, and canvas element. Exports `renderer`, `scene`, and `canvas`.
- **camera.ts**: PerspectiveCamera setup with window resize handling. Exports `camera`.
- **gui.ts**: Tweakpane debug panel with FPS graph monitor. Exports `pane` and `fpsGraph`.
- **orbit-control.ts**: OrbitControls for camera interaction with damping. Exports `controls`.

### Main Application (src/main.ts)

Entry point that:

- Sets up lighting (ambient + directional with shadows)
- Creates geometries and materials
- Implements shader-based animated sphere using custom GLSL
- Runs the animation loop updating shader uniforms (`uTime`, `uFrequency`)

### Shaders (src/shaders/)

GLSL files loaded via `vite-plugin-glsl`. Import as strings:

```typescript
import vertexShader from '/@/shaders/vertex.glsl'
```

## Key Patterns

- **Functional architecture**: No classes. Core modules export initialized objects directly.
- **Path alias**: `/@/` maps to `./src/` for cleaner imports
- **Real-time animation**: Uses Three.js Clock for delta time in the render loop
- **Debug UI binding**: Tweakpane bound directly to object properties (camera position, light settings)
- **Shader uniforms**: Updated each frame via `material.uniforms.uTime.value`

## TypeScript Configuration

- `moduleResolution: "bundler"` (Vite-optimized)
- Strict mode with `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`
- Types included: `vite/client`, `three`, `tweakpane`, `three/tsl`, `three/webgpu`

## Code Style

- Uses `@alvarosabu/eslint-config` (no semicolons)
- ES modules throughout
- Canvas element requires `id="webgl"` in index.html

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, but it invokes Vite through `vp dev` and `vp build`.

## Vite+ Workflow

`vp` is a global binary that handles the full development lifecycle. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

### Start

- create - Create a new project from a template
- migrate - Migrate an existing project to Vite+
- config - Configure hooks and agent integration
- staged - Run linters on staged files
- install (`i`) - Install dependencies
- env - Manage Node.js versions

### Develop

- dev - Run the development server
- check - Run format, lint, and TypeScript type checks
- lint - Lint code
- fmt - Format code
- test - Run tests

### Execute

- run - Run monorepo tasks
- exec - Execute a command from local `node_modules/.bin`
- dlx - Execute a package binary without installing it as a dependency
- cache - Manage the task cache

### Build

- build - Build for production
- pack - Build libraries
- preview - Preview production build

### Manage Dependencies

Vite+ automatically detects and wraps the underlying package manager such as pnpm, npm, or Yarn through the `packageManager` field in `package.json` or package manager-specific lockfiles.

- add - Add packages to dependencies
- remove (`rm`, `un`, `uninstall`) - Remove packages from dependencies
- update (`up`) - Update packages to latest versions
- dedupe - Deduplicate dependencies
- outdated - Check for outdated packages
- list (`ls`) - List installed packages
- why (`explain`) - Show why a package is installed
- info (`view`, `show`) - View package information from the registry
- link (`ln`) / unlink - Manage local package links
- pm - Forward a command to the package manager

### Maintain

- upgrade - Update `vp` itself to the latest version

These commands map to their corresponding tools. For example, `vp dev --port 3000` runs Vite's dev server and works the same as Vite. `vp test` runs JavaScript tests through the bundled Vitest. The version of all tools can be checked using `vp --version`. This is useful when researching documentation, features, and bugs.

## Common Pitfalls

- **Using the package manager directly:** Do not use pnpm, npm, or Yarn directly. Vite+ can handle all package manager operations.
- **Always use Vite commands to run tools:** Don't attempt to run `vp vitest` or `vp oxlint`. They do not exist. Use `vp test` and `vp lint` instead.
- **Running scripts:** Vite+ built-in commands (`vp dev`, `vp build`, `vp test`, etc.) always run the Vite+ built-in tool, not any `package.json` script of the same name. To run a custom script that shares a name with a built-in command, use `vp run <script>`. For example, if you have a custom `dev` script that runs multiple services concurrently, run it with `vp run dev`, not `vp dev` (which always starts Vite's dev server).
- **Do not install Vitest, Oxlint, Oxfmt, or tsdown directly:** Vite+ wraps these tools. They must not be installed directly. You cannot upgrade these tools by installing their latest versions. Always use Vite+ commands.
- **Use Vite+ wrappers for one-off binaries:** Use `vp dlx` instead of package-manager-specific `dlx`/`npx` commands.
- **Import JavaScript modules from `vite-plus`:** Instead of importing from `vite` or `vitest`, all modules should be imported from the project's `vite-plus` dependency. For example, `import { defineConfig } from 'vite-plus';` or `import { expect, test, vi } from 'vite-plus/test';`. You must not install `vitest` to import test utilities.
- **Type-Aware Linting:** There is no need to install `oxlint-tsgolint`, `vp lint --type-aware` works out of the box.

## Review Checklist for Agents

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to validate changes.
<!--VITE PLUS END-->

---
> Source: [alvarosabu/modern-three](https://github.com/alvarosabu/modern-three) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
