## sigurd-startup-game

> You are working on **Sigurd Startup**, a 2D arcade platformer shipped as an npm package (a Web Component + React) and embedded in a separate React landing page with Stripe + Firebase. This document is the contract for how you operate in this repo. Read it every session.

# CLAUDE.md — Sigurd Startup

You are working on **Sigurd Startup**, a 2D arcade platformer shipped as an npm package (a Web Component + React) and embedded in a separate React landing page with Stripe + Firebase. This document is the contract for how you operate in this repo. Read it every session.

---

## 1. Project overview

- **Game:** 2D arcade platformer (Bomb Jack–inspired), 800×600 fixed playfield, 60 FPS target
- **Genre mechanics:** bomb collection in sequence + coin power-ups + monster avoidance
- **Engine:** a **custom canvas 2D engine** — a hand-rolled game loop (`GameLoopManager`) rendering to a `<canvas>` via `RenderManager`. **There is no Phaser and no Matter.js.** Do not add them.
- **Language:** TypeScript, `strict: true`
- **Distribution:** published to npm as `sigurd-startup`, consumed by a separate React landing page
- **Delivery form:** a Web Component `<sigurd-startup>` (registered in `src/game-wrapper.tsx`) that mounts a React tree; also importable in React
- **Host integration:** the landing page provides a `window.sigurdGame` bridge (balance, credits, Stripe, Firebase, leaderboard)
- **Authoritative design source:** `game-specs.md` at the repo root — the source of truth for all mechanics, constants, timings, and behaviors. Code and commits cite it as `game-specs §X`.

> ⚠️ There is a second, **divergent** spec copy at `specs/game-spec.md.md` (note the doubled extension). It is stale. Treat `game-specs.md` (root) as canonical until the two are reconciled.

## 2. Before you write code

**Always, in this order:**

1. Read the relevant section of `game-specs.md` before implementing any gameplay system. If the spec is ambiguous, **ask** — do not guess.
2. Check `DECISIONS.md` for architectural choices already made. Do not re-litigate them. (Note: its oldest entries describe an abandoned Phaser design — see the correcting entry at the top.)
3. Propose a plan before writing code for any task that touches more than two files. Wait for approval.
4. When implementing, reference the spec section by number in commit messages (e.g., "implement coin spawn conditions per game-specs §7").

If you find yourself guessing at a constant, a timing, or a behavior, stop and check the spec.

## 3. Repo layout

```
src/
  managers/         # cross-entity coordinators — the heart of the game loop
    GameLoopManager.ts    # requestAnimationFrame loop, fixed-step orchestration
    RenderManager.ts      # draws the playfield to <canvas>
    CollisionManager.ts   # custom AABB collision + resolution
    GameManager.ts / GameStateManager.ts   # top-level orchestration + state
    bombManager.ts, coinManager.ts, coinPhysics.ts
    ScalingManager.ts, PowerUpManager.ts, ScoreManager.ts, AudioManager.ts
    MonsterFactory.ts, MonsterBehaviorManager.ts, monster-movements/
    Optimized{Spawn,Respawn}Manager.ts, InputManager.ts, LevelManager.ts, ...
  stores/           # Zustand stores — the single source of truth for state
    game/           # scoreStore, stateStore, levelStore
    entities/       # playerStore, coinStore, monsterStore
    systems/        # audioStore, inputStore, renderStore, balanceStore, tuningStore
    gameStore.ts    # root store composition
  entities/         # plain game-object classes (Player, Bomb, monster subclasses)
  lib/              # pure logic + helpers (gravityLUT, scoringUtils, gameBridge, logger, ...)
  components/       # React UI: menus/HUD/overlays positioned over the canvas
    menu/menus/     # StartMenu, PauseMenu, BonusScreen, GameOverScreen, ...
  config/           # tunable constants, grouped by concern (see §4)
  types/            # interfaces.ts, enums.ts, constants.ts (deprecated shim)
  editor/           # in-repo level/tuning editor (React)
  maps/             # map definitions
  tutorials/        # tutorial mission definitions
  game-wrapper.tsx  # Web Component registration (customElements.define)
  index.ts          # npm package entry point (non-React exports + side-effect registration)
game-specs.md       # design spec (READ THIS) — canonical
DECISIONS.md        # architectural log (append-only)
CLAUDE.md           # this file
```

- **The canvas renders the playfield only.** Menus, HUD chrome, settings, countdown, bonus screen are React components in `src/components/` positioned over the canvas.
- **Game logic lives in managers and pure `lib/` functions, not React.** Managers own the loop and coordinate; `lib/` holds pure helpers; React owns chrome.

## 4. Naming and style rules

- Files: `kebab-case.ts` for logic (e.g., `coin-physics`), but note much of the existing tree uses `camelCase.ts` (`bombManager.ts`) and `PascalCase.ts` for classes (`ScalingManager.ts`). **Match the convention of the directory you're editing.**
- Classes: `PascalCase`. Methods/variables: `camelCase`.
- Constants: `UPPER_SNAKE_CASE`, grouped by concern in **`src/config/*.ts`** (e.g., `config/coins.ts`, `config/scoring.ts`, `config/game.ts`). `src/types/constants.ts` is a **deprecated** backwards-compat shim (`GAME_CONFIG`) — new code should import from `@/config`.
- Enum/event tokens: see `src/types/enums.ts` (e.g., `PauseReason`, `AudioEvent`, `MonsterType`).
- One class per file. Prefer named exports.

**TypeScript:**
- `strict: true` is non-negotiable.
- **Goal: no `any`.** There is currently a backlog of `@typescript-eslint/no-explicit-any` violations, so `npm run lint` is **non-blocking in CI** for now — do not add new `any`, and clear existing ones when you touch a file.
- No `@ts-ignore` without a comment explaining why.
- Prefer `readonly` and `const`. Use discriminated unions for state (e.g., monster behavior, `PauseReason`).
- Public APIs get JSDoc. Internals don't need it.

## 5. Architecture rules

- **State lives in Zustand stores** (`src/stores/`), one store per concern. Score in `scoreStore`, coins in `coinStore`, etc. No duplicated state.
- **Managers coordinate; they do not each own a private copy of shared state.** A manager reads/writes the relevant store and exposes methods; the loop drives managers each frame.
- **There is no `EventBus` singleton** (an earlier design assumed one; it was never built). Cross-system communication goes through stores and explicit manager method calls. Do not introduce a global singleton without asking.
- **Entities do not contain business logic.** A `Bomb` knows its position and render/collision shape; `BombManager` decides what collecting it means.
- **Pure logic stays pure.** Anything in `src/lib/` that is a calculation (scoring, gravity LUT, spawn math) takes inputs and returns outputs with no side effects — trivial to test.

## 6. Do NOT do these things without asking

- Add new npm dependencies (especially game/physics libs — no Phaser, no Matter.js)
- Modify `package.json` `exports`, the package entry point (`src/index.ts` / `game-wrapper.tsx`), or the build config (`vite.lib.config.ts`)
- Change the `window.sigurdGame` bridge contract (`SigurdGameBridge` in `src/lib/gameBridge.ts`)
- Introduce a global singleton
- Change the Web Component tag name or scene/screen structure
- Modify design values in `game-specs.md` — that's a design decision, not a code decision
- Touch `src/components/` (UI) when the task is gameplay-only, or vice versa

## 7. Physics rules (critical)

Per `game-specs §4`, physics values are **per-frame at 60 FPS**. The loop delivers a `delta` in ms.

- Multiply per-frame values by `(delta / 16.67)` for frame-rate-independent motion (see `coinPhysics.ts`, `advanceGravityIndex` in `lib/gravityLUT.ts`).
- **Player gravity uses a lookup table**, not naive acceleration: `src/lib/gravityLUT.ts` (128-entry curve with an apex hang and terminal-velocity cap, per `game-specs §4.3`). Advance the index by frame; read velocity from the LUT.
- Collision detection/resolution is **custom AABB** in `CollisionManager` — check all platforms, resolve by smallest penetration.
- Boundary collision: left/right/top clamp; bottom (below `PLAYFIELD_BOTTOM`) is player death.
- Coin physics are per-type (`config/coinTypes.ts` → `COIN_PHYSICS`): STANDARD bounces under gravity with damping; POWER reflects with no energy loss and ignores gravity; GRAVITY_ONLY (B/M/F) falls straight with edge-fall. Logic lives in `managers/coinPhysics.ts`.

## 8. Testing

- Framework: **Vitest**. Tests live next to the code as `*.test.ts` (e.g., `src/lib/gravityLUT.test.ts`).
- **Test pure logic, not rendering.** Good candidates: scoring math (`lib/scoringUtils.ts`), multiplier thresholds, bomb-sequence validation (`managers/bombManager.ts`), coin physics (`managers/coinPhysics.ts`), difficulty scaling, gravity LUT, A* / pathfinding.
- Do NOT write tests that boot the full engine or the canvas. Too slow, too flaky.
- Run tests: `npm test` (`vitest run`). Watch mode: `npm run test:watch`.
- Always run tests before declaring a task complete. If you add logic in `src/lib/` or `src/managers/`, add tests.

## 9. Running the game & scripts

- Dev server / standalone harness: `npm run dev` → `http://localhost:5173`
- Build the app: `npm run build`
- Build the publishable library: `npm run build:lib` (config: `vite.lib.config.ts`)
- Package + smoke test: `npm run build:package`
- Typecheck: `npx tsc --noEmit`
- Lint: `npm run lint` (currently non-blocking — see §4)

## 10. The bridge (`window.sigurdGame`)

Contract per `game-specs §16` and `src/lib/gameBridge.ts`. **Do not change this API without approval.**

```ts
interface SigurdGameBridge {
  ready: boolean;
  getBalance(): number;
  deductCredits(amount: number): Promise<{ success: boolean; newBalance: number; error?: string }>;
  refreshBalance(): Promise<number>;
  onBalanceChanged(cb: (info: BalanceInfo) => void): () => void;
  sendGameCompletion(data: unknown): void;
  sendAudioSettings(settings: unknown): void;
  loadUserAudioSettings(userId: string): Promise<void>;
  grantBusinessIdea(amount: number): void;   // F-coin reward; host validates server-side
  openPurchase?(): void;                       // optional; package falls back to same-tab nav
  openLeaderboard?(): void;                     // optional; same contract as openPurchase
}
```

- All bridge interaction goes through the helpers in `src/lib/gameBridge.ts` (`waitForBridge`, `hasBridge`, `getBalance`, `deductCredits`, `subscribeBalance`, `openPurchasePage`, `openLeaderboardPage`). Prefer these over touching `window.sigurdGame` directly elsewhere.
- Detection timeout is `BRIDGE_TIMEOUT_MS = 3000`; when no host bridge is present the package degrades gracefully (standalone behavior).
- Credit deductions: always `await` the result and block the round start on failure.

## 11. Assets

- Sprites live in `src/assets/`; per-map backgrounds in `src/assets/maps-bg-images/`.
- Audio is **file-based** in `src/assets/audio/`: background music as `.mp3`, SFX as `.wav` (`jump.wav`, `coin-catch.wav`, `monster-kill.wav`, `Victory.wav`, `gameover.wav`, …). Audio is coordinated by `AudioManager` + `audioStore`.
- When adding new art, reuse existing pixel dimensions (player 25×35; bomb/coin/monster 25×25). Don't rescale.

## 12. Workflow expectations

- **Plan before anything non-trivial.** Propose the approach for multi-file work and wait for approval.
- **Commit after every working feature**, not at the end of a session. Commit format: `<area>: <what> (game-specs §X)`. Example: `coins: implement P-coin color cycle (game-specs §7.1)`.
- **Ask before broad refactors.** Propose restructuring as a separate task — don't bundle with a feature.
- **Paste errors back.** If the game throws at runtime, capture the stack or check `npm run dev` output.
- **Stop at uncertainty.** If the spec is ambiguous on a detail, ask. Don't pick a direction silently.

## 13. Common anti-patterns — do not do these

- Hardcoding numbers that exist in the spec. Use `src/config/*`.
- Mixing React state and engine state. React owns menu/HUD chrome; the engine/stores own the playfield. They sync via Zustand stores and props.
- Putting gameplay logic inside React components.
- Using `console.*` directly. Use `src/lib/logger.ts` (`log` / `logger`) so logging respects categories and throttling.
- Using `setTimeout`/`setInterval` for gameplay timing. Drive timing off the game loop's `delta` so pause/resume is respected.
- Forgetting that power mode pauses difficulty scaling (`ScalingManager` + `PauseReason.PowerMode`). Any time-based system must check pause state.
- Creating new monsters by copying existing classes wholesale. Extend the base monster and reuse `monster-movements/`.
- Adding Phaser, Matter.js, or a global `EventBus` — none of these exist here.

## 14. When in doubt

Ask. A 30-second question saves a 30-minute revert.

---

*Last updated: 2026-08-06 — rewritten to match the actual custom-canvas + Zustand + React architecture (the previous version described an abandoned Phaser design). Update this file when architectural decisions change.*

---
> Source: [jonasanders1/sigurd-startup-game](https://github.com/jonasanders1/sigurd-startup-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
