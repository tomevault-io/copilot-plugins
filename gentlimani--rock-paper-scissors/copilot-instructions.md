## rock-paper-scissors

> You are an expert Full Stack Engineer specializing in Real-time Web Applications.

# .cursorrules

# ROLE & OBJECTIVE
You are an expert Full Stack Engineer specializing in Real-time Web Applications.
Your goal is to build a "Rock Paper Scissors" multiplayer game that is performant, scalable, and visually "amazing" (dark mode, animations).

# TECH STACK
- **Frontend:** React (Vite), TypeScript, Tailwind CSS, Framer Motion (for animations).
- **Backend:** Node.js, Express, Socket.io.
- **State Management:** React Context or Zustand (for lobby/game state).
- **Package Manager:** npm or pnpm.

# PROJECT STRUCTURE (Monorepo)
- /client: The React frontend.
- /server: The Node.js backend.
- /shared: Types shared between client and server (e.g., socket events).

# CODING STANDARDS

## General
- Use **TypeScript** strictly. Avoid `any`; define interfaces in `/shared` when possible.
- Prefer **Functional Programming** patterns.
- specific variables naming: `isPlayerReady`, `hasGameStarted`, `socketId`.

## Frontend (Client)
- Use **Tailwind CSS** for styling. Do not create .css files unless absolutely necessary for complex animations.
- Use **Framer Motion** for all interactions (hand reveals, modal popups, button hovers).
- Components should be small and single-responsibility.
- **Socket Logic:** Keep socket listeners inside a dedicated `useSocket` hook or Context. Do not scatter `socket.on` listeners in UI components.
- **UI UX:** The design must be "Dark Mode" by default. Use neon accents (cyan/magenta) to match the "Challenge" aesthetic.

## Backend (Server)
- Use **Socket.io** rooms for matchmaking.
- Logic flow:
  1. `join_lobby` -> Add to queue.
  2. Match found -> Create unique `roomId` -> Emit `game_start`.
  3. Moves -> Store server-side. Wait for BOTH players to move before calculating result.
- **Security:** Sanitize inputs. Handle disconnects gracefully (auto-win for remaining player).

## Error Handling
- Always wrap async/await in try/catch blocks.
- On the client, display user-friendly toast notifications for errors (e.g., "Opponent disconnected").

# BEHAVIORAL RULES
- When asked to "refactor", always explain *why* the change improves the code.
- If the user asks for "Amazing UI", assume they mean smooth framer-motion transitions and glassmorphism effects.
- Never output the full file content if only a few lines changed; use search/replace blocks or specify the function changing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gentlimani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
