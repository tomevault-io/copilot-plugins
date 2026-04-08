## tabtriage

> This document provides guidelines for the Gemini AI to follow when working on this project.

# GEMINI.md - Guidelines for Gemini

This document provides guidelines for the Gemini AI to follow when working on this project.

## Build Commands
- `pnpm dev` - Start development server
- `pnpm build` - Create production build
- `pnpm start` - Run production build
- `pnpm lint` - Run ESLint
- `pnpm migrate` - Run database migrations
- `pnpm db:push` - Push database changes using Drizzle Kit
- `pnpm db:studio` - Open Drizzle Studio

## Code Style
- **TypeScript**: Use strict typing. Define types and interfaces in `/src/types`.
- **Imports**: Use path aliases (`@/*`) for all imports.
- **Components**: Use PascalCase for component files and functions.
- **Variables**: Use camelCase for variables and functions.
- **Error Handling**: Use try/catch blocks and the logger utility at `/src/utils/logger.ts`.
- **Formatting**: Adhere to the existing formatting (2-space indentation, semicolons).
- **Logging**: Use the logger at `/src/utils/logger.ts` with appropriate log levels.
- **File Organization**: Follow Next.js App Router conventions.
- **State Management**: Use React Query and Zustand for state management.
- **Environment Variables**: Use `.env.local` for local development, and validate variables in the code.
- **Testing**: No test framework is currently defined.
- **Package Manager**: Always use `pnpm`. Do not use `npm` or `yarn`.

## Architecture
- **Framework**: Next.js with the App Router.
- **Database**: Drizzle ORM with a Turso/libSQL database.
- **AI Features**: OpenAI and Hugging Face Transformers are used.
- **Styling**: Tailwind CSS.
- **File Uploads**: UploadThing is used for file uploads.
- **Authentication**: Handled through Next.js middleware and context providers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nibzard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nibzard)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
