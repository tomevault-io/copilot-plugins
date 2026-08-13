## devagents

> A professional AI Software Engineer workspace featuring an autonomous 3D agent, a virtual developer environment, and a sophisticated reasoning engine. The system integrates real-time work visualization, persistent memory via Supabase, and advanced code analysis capabilities.

## Project Summary
A professional AI Software Engineer workspace featuring an autonomous 3D agent, a virtual developer environment, and a sophisticated reasoning engine. The system integrates real-time work visualization, persistent memory via Supabase, and advanced code analysis capabilities.

## Tech Stack
- **Framework**: Next.js 15 (App Router)
- **Frontend**: React, Three.js, React Three Fiber, Tailwind CSS
- **Icons**: Lucide React
- **Database**: Supabase
- **Language**: TypeScript

## Architecture
- `src/app`: Next.js pages and API routes (`/api/agent`, `/api/thinking`, etc.)
- `src/components`: UI components including `Room3D`, `ComputerScreen`, and specialized panels.
- `src/lib`: Core logic for the AI engineer, state management, and Supabase integration.
- `chat_history.md`: Contextual log of project progress and previous chat history.

## User Preferences
- **Context Management**: AI coder must read `chat_history.md` before starting any work to understand project state.
- **Aesthetics**: Professional AI engineer workspace with a dark, sophisticated theme (JetBrains Mono font, slate/cyan/purple palette).

## Project Guidelines
- Follow the reasoning loop: Observe -> Reflect -> Plan -> Execute.
- Maintain consistency in UI components (backdrop-blur, border styles, Lucide icons).
- Keep `chat_history.md` updated with major project milestones.

## Common Patterns
- API routes for agent behavior in `src/app/api/`.
- Client-side state management for the workspace in `src/lib/workspaceStore.ts`.

---
> Source: [devagents-os/devagents](https://github.com/devagents-os/devagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
