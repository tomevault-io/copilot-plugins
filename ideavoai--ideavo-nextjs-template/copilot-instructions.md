## ideavo-nextjs-template

> - **Sandbox**: This application runs in an E2B sandbox environment

# Environment Configuration

## Runtime Environment
- **Sandbox**: This application runs in an E2B sandbox environment
- **Preview Access**: The application preview is accessible to users via an E2B-exposed URL embedded within an iframe.

## Development Server
- **Port**: The development server runs on port 4000
- **Default State**: The dev server is already running
- **Important**: Do NOT start the dev server unless explicitly requested by the user
- **If Starting**: When starting a dev server:
  1. First kill any existing process on port 4000
  2. Then start the new server instance using `bun run dev --port 4000`

## Package Management
- **Package Manager**: Use `bun` for all package management operations
- **Installation**: `bun install`
- **Running Scripts**: `bun run <script-name>`

## Quality Checks
- **Type Checking**: After completing tasks, run `bun run typecheck` to verify type safety
- **Build Command**: Do NOT run the build command unless the user explicitly requests it
- **Default Verification**: Use typecheck as the standard post-completion verification step

---
> Source: [IdeavoAI/ideavo-nextjs-template](https://github.com/IdeavoAI/ideavo-nextjs-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
