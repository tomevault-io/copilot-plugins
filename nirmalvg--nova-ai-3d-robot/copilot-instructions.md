## nova-ai-3d-robot

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

# Project Nova: AI Agent Instructions

## Project Overview

Project Nova is a premium, browser-native 3D AI humanoid robot companion. It blends high-end sci-fi web UI (glassmorphism, cinematic lighting) with fully articulated 3D character animations and voice-to-voice LLM interactions.

## Core Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS v4
- **3D Engine:** Three.js + @react-three/fiber (R3F) + @react-three/drei
- **State Management:** Zustand (for global state), React local state
- **AI/Web APIs:** Web Speech API (STT/TTS), OpenAI/Claude (LLM)
- **Icons:** Lucide React

## Architecture Philosophy

The application is strictly divided into two visual layers that must not interfere with each other:

1. **The WebGL Canvas (`src/components/NovaScene`):** The 3D world containing the robot, lighting, and environment.
2. **The DOM Overlay (`src/components/nova-ui`):** The 2D UI layered on top of the canvas using absolute positioning and `z-index`.

### Agent Directives & Coding Standards

#### 1. Next.js 16 App Router Rules

- By default, assume components are Server Components.
- You MUST explicitly add `"use client";` at the very top of any file that uses React Hooks (`useState`, `useEffect`, etc.), Browser APIs, or `@react-three/fiber` components.
- Keep the `app/` directory lean. Logic and heavy lifting should live in `src/components`, `src/hooks`, or `src/lib`.

#### 2. React Three Fiber (3D) Best Practices

- **Performance is critical.** Minimize re-renders in the `useFrame` hook.
- Do not instantiate new objects (e.g., `new THREE.Vector3()`) inside `useFrame`. Declare them outside the hook and reuse them.
- Use `useMemo` for computationally expensive Geometries or Materials.
- Separate 3D components logically (e.g., `<Robot />`, `<EnvironmentLights />`, `<CameraRig />`).
- All 3D assets (`.glb`, `.gltf`) live in the `public/` directory and should be loaded via `@react-three/drei`'s `useGLTF`.

#### 3. Styling & UI (Tailwind v4)

- Maintain the "Premium Sci-Fi" aesthetic: Glassmorphism, deep dark backgrounds (`bg-neutral-950`), glowing accents (`cyan-500`), and minimalist typography.
- Use Tailwind utility classes directly in the `className` attribute.
- For complex glassmorphism, use combinations of `backdrop-blur-xl`, `bg-black/40`, and thin borders `border-white/10`.
- Ensure the center of the screen remains clear of heavy UI elements so the 3D robot is always visible.

#### 4. Agent Behavior & Responses

- **Think step-by-step:** When writing complex 3D math (like quaternions, inverse kinematics, or raycasting), briefly explain the logic before providing the code.
- **Provide complete files:** When modifying a component, provide the complete, updated code block rather than "insert this snippet here" to avoid syntax errors.
- **TypeScript First:** Ensure all props, state, and API responses are properly typed. Use interfaces for all data structures.
- **Do not break the 4th wall:** Maintain a collaborative, builder-focused tone.

## File Structure Reference

```text
project-nova/
├── public/                 # 3D models (.glb), audio clips, static assets
├── src/
│   ├── app/                # Next.js App Router (page.tsx, layout.tsx)
│   ├── components/
│   │   ├── nova-ui/        # 2D DOM Overlay components (HUD, Docks)
│   │   └── NovaScene/      # 3D R3F components (Canvas, Models, Lights)
│   ├── hooks/              # Custom React/R3F hooks (e.g., useSpeech, useBrain)
│   ├── store/              # Zustand global state stores
│   └── lib/                # Utility functions, AI API wrappers
├── tailwind.config.ts
└── AGENT.md                # This file
<!-- END:nextjs-agent-rules -->
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NirmalVG)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NirmalVG)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
