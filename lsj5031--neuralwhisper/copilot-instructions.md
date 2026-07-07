## neuralwhisper

> - **Dev server**: `npm run dev` (runs on port 3000)

# NeuralWhisper Agent Guidelines

## Build & Development

- **Dev server**: `npm run dev` (runs on port 3000)
- **Build**: `npm run build` (produces dist/)
- **Preview**: `npm run preview`
- **No test script**: This is a frontend-only React app without unit tests

## Architecture

**Frontend-only React + TypeScript + Vite + Tailwind**
- **Entry**: `index.tsx` → `App.tsx`
- **Components** (`components/`): Reusable UI (CyberUI, TranscriptionForm, TaskList, SettingsModal)
- **Services** (`services/apiService.ts`): REST API wrapper for audio transcription service
- **Types** (`types.ts`): TypeScript interfaces (ApiConfig, TranscriptionRequest, TaskStatus, etc.)
- **Styling**: Tailwind CSS with custom cyberpunk theme (cyber-cyan, cyber-black, etc.)

**API Integration**: Communicates with external Whisper API (configurable base URL, admin key auth via `x-admin-api-key` header)

## Code Style

- **Language**: TypeScript (strict mode, React JSX)
- **Import paths**: Use `@/` alias for absolute imports (configured in tsconfig.json)
- **Components**: Functional React with hooks, lowercase filenames (PascalCase for exports)
- **Styling**: Inline Tailwind classes (prefer semantic Tailwind utility classes)
- **Error handling**: `try-catch` blocks, propagate errors as thrown exceptions
- **Naming**: camelCase for functions/variables, PascalCase for components/types, SCREAMING_SNAKE_CASE for constants
- **Exports**: Named exports for components, default export for App
- **State**: React hooks (useState, useEffect); persist UI config to localStorage

---
> Source: [lsj5031/NeuralWhisper](https://github.com/lsj5031/NeuralWhisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
