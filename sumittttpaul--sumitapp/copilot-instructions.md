## sumitapp

> Guidance for Cursor Agent working in the SumitApp repo; auto-attaches for CLI, templates, and official-site directories.


# SumitApp Cursor Rules

Purpose: Define project-specific rules and standards for Cursor when working in this repository.

Project context

- Monorepo tooling: Turborepo (in templates), Bun-first scripts
- Apps in templates: Next.js website, Expo mobile, Node backend
- This repo contains CLI (create-sumit-app), official-site, and templates used to scaffold new projects

TypeScript

- Strict mode across packages
- Explicit return types for public functions
- Use interface for shapes; type for unions/intersections
- Use import type for type-only imports
- Prefer narrow types and const assertions where applicable

Package manager

- Primary: Bun ("packageManager": "bun@1.2.22")
- Use bun run for scripts, bun --cwd for workspace-specific scripts
- Use workspace:^ protocol between internal packages (in templates)

CLI development (create-sumit-app)

- Use commander for command parsing
- Use prompts for interactive input with validation
- Use ora for progress; chalk for colorized output
- Use degit for template cloning (no git history)
- Support --verbose flag and non-interactive flows
- Clear, actionable error messages with proper exit codes

Template development (templates/default)

- Preserve Turborepo task graph and caching
- Keep shared packages under packages/, apps under projects/
- Ensure workspace dependencies use workspace:^ and build order is correct
- Verify template generates and runs with bun dev/build/lint/check-types

Next.js/Expo conventions

- Next.js 16 App Router and React 19 types
- Prefer Server Components where possible
- Tailwind CSS 4 with utility-first approach
- Expo 54 for mobile, NativeWind, React Native Paper

Testing

- Vitest for CLI; unit + integration tests
- Test template generation and basic script execution
- Maintain reasonable coverage; name tests descriptively

Error handling and logging

- Use structured logger with levels: verbose, info, warn, error, success
- Include stack traces only in verbose mode
- Add retry for transient FS/network errors on Windows

Security and performance

- Sanitize user/project names and file paths
- Avoid heavy imports on CLI startup; lazy-load where feasible
- Stream/copy efficiently for large template operations

Review checklist

- Affects generated projects? Test end-to-end
- Cross-platform checks (Windows/macOS/Linux)
- Docs updated (README/WARP/official-site if needed)
- Types accurate; scripts runnable with Bun

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sumittttpaul) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
