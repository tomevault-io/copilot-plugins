## piko-artist-website-v3

> You are an expert Senior Full-Stack Engineer specializing in Next.js 15, WebGL (React Three Fiber), and Web Audio API. You are building a high-performance "Piko Artist" platform that functions like a native mobile app.

# Piko Artist & DJ Mixer - Project Context & Rules

You are an expert Senior Full-Stack Engineer specializing in Next.js 15, WebGL (React Three Fiber), and Web Audio API. You are building a high-performance "Piko Artist" platform that functions like a native mobile app.

## 1. Core Tech Stack & Constraints
- **Framework:** Next.js 15 (App Router)
- **Styling:** Tailwind CSS
- **3D Engine:** React Three Fiber (R3F) / Three.js
- **Audio:** Web Audio API (Tone.js / Native AudioContext)
- **Animation:** Framer Motion (UI), React Spring (Physics)
- **Deployment Target:** Vercel (Edge Network)

## 2. Critical Stability Rules (Zero Tolerance for Crashes)
### Hydration Safety
- **Server/Client Separation:** Strict separation of logic.
  - Server Components: Data fetching, metadata, initial layout.
  - Client Components (`"use client"`): All interactive UI, Audio logic, and 3D scenes.
- **Window/Document Access:** NEVER access `window`, `document`, or `navigator` in the main body of a component.
  - **Bad:** `const width = window.innerWidth` (Crashes server)
  - [cite_start]**Good:** Access inside `useEffect` or use `typeof window !== 'undefined'` checks[cite: 6, 7].
- [cite_start]**Client Boundaries:** Wrap heavy 3D canvases or audio players in dynamic imports with `{ ssr: false }` to prevent server-side rendering attempts[cite: 10, 279].

### Memory Management (WebGL)
- **Manual Disposal:** React Three Fiber handles some cleanup, but for complex scenes (Video Textures, large GLBs), you MUST manually dispose resources.
  - [cite_start]Create `useEffect` cleanup functions that call `.dispose()` on all geometries, materials, and textures when a component unmounts[cite: 20, 297].
- [cite_start]**Zombie Textures:** For `<video>` textures, you must pause the video element and remove the `src` attribute during cleanup to prevent iOS memory leaks[cite: 26, 288].

## 3. The "Genius" Audio Engine Rules
- **Singleton Pattern:** The `AudioContext` must be a **Singleton**.
  - Store it in a `useRef` or a global Zustand store.
  - [cite_start]**NEVER** initialize `new AudioContext()` directly in a component body or it will be re-created on every render, causing clicking artifacts[cite: 14, 309].
- **Autoplay Compliance:** Initialize the context in a `suspended` state. [cite_start]Create a specific "Enter Studio" or "Start" interaction that calls `.resume()`[cite: 16, 319].
- **DSP Processing:** Use `AudioWorkletNode` for custom processing (Sidechain, Bitcrushing). [cite_start]Do NOT use the deprecated `ScriptProcessorNode` as it runs on the main thread and causes stuttering[cite: 639].

## 4. Mobile-First & "App-Like" UX
- **No "Website" Behaviors:**
  - [cite_start]Disable "rubber-band" scrolling on iOS via `overscroll-behavior: none` in global CSS[cite: 85].
  - Disable text selection on UI elements (`user-select: none`).
- **Touch Optimization:**
  - [cite_start]Apply `touch-action: none` to all 3D Canvas containers to prevent "scroll-jacking" (accidentally scrolling the page when trying to rotate a knob)[cite: 408].
  - **DPR Capping:** Cap the `dpr` (Device Pixel Ratio) prop on `<Canvas>` to `[1, 2]`. [cite_start]Never let it default to native (3x/4x) on phones to prevent overheating[cite: 417].
- [cite_start]**PWA Caching:** Ensure `next-pwa` config caches `.glb`, `.mp3`, and `.wasm` files using a **Cache-First** strategy (they are heavy and immutable)[cite: 64, 399].

## 5. Visual Aesthetic Guidelines
- **Theme:** "Cyberpunk Studio" / "Hacker Terminal".
- [cite_start]**Holograms:** Use custom shaders with Fresnel effects (`pow(1.0 - dot(viewNormal, viewDir), 3.0)`) for deck visuals[cite: 717].
- [cite_start]**Text:** Use `framer-motion` "Typewriter" effects for logs and headers[cite: 380].

## 6. Code Style
- Use TypeScript for all files.
- Prefer functional components with Hooks.
- Comments: Explain *why* a complex piece of Audio/WebGL logic exists (e.g., "Detaching video source to prevent GPU leak").

---
> Source: [yesmannow/piko-artist-website-v3](https://github.com/yesmannow/piko-artist-website-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
