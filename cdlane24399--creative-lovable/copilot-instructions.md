## creative-lovable

> - When creating new components, place them in the `components/` directory, organized by feature.

# .cursorrules - Configuration for Cursor AI

# Ignore these files and directories
[ignore]
.git/
node_modules/
.next/
out/
.vercel/
.DS_Store
Thumbs.db
.idea/
.vscode/
*.swp
*.swo
*.tsbuildinfo
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*
coverage/
dist/
build/
.genkit/
.svggen/
.cursor/debug.log

# Define project-specific rules
[rules]

# General
- When creating new components, place them in the `components/` directory, organized by feature.
- Use the `cn` utility from `lib/utils.ts` for conditional class names.
- Follow the existing code style and formatting.

# API Routes
- All API routes should be placed in the `app/api/` directory.
- Use the `withAuth` middleware for authenticated routes.
- Use the `withRateLimit` middleware for all public-facing API routes.
- Use the structured logger from `lib/logger.ts` for all logging.

# Components
- Use Radix UI primitives for building accessible components.
- Use Tailwind CSS for styling.
- Use TypeScript for all components and define props with interfaces.

# AI Integration
- Use the `createContextAwareTools` function to create tools for the AI agent.
- Use the `generateAgenticSystemPrompt` function to create the system prompt for the AI agent.
- All AI-related logic should be in the `lib/ai/` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cdlane24399) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
