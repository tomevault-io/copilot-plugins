## rivers-of-reckoning

> > **An immersive, procedurally generated roguelike RPG built with Strata for web and mobile**

# CLAUDE.md - Rivers of Reckoning

> **An immersive, procedurally generated roguelike RPG built with Strata for web and mobile**

## 🌊 The Vision

**Rivers of Reckoning** is a 2.5D side-scrolling adventure where players explore an infinite, ever-changing world of marshes, forests, deserts, and tundra. Every playthrough is unique—generated from a seed that creates coherent biomes, dynamic weather, and challenging encounters.

### Player Experience Goals

1. **Instant Play**: Click/tap and you're in. No downloads, no waiting.
2. **One More Turn**: Addictive exploration loop - "what's over that next hill?"
3. **Tactile Feedback**: Responsive controls, satisfying combat, haptic feedback on mobile
4. **Mobile-First**: Touch controls that feel native with Strata triggers
5. **Shareable Worlds**: Share your seed with friends to explore the same world
6. **Persistent Progress**: LocalStorage/Capacitor saves your best runs and achievements

### The World

- **Biomes** flow naturally into each other based on temperature and moisture
- **Weather** changes dynamically - rain slows you, storms are dangerous
- **Day/Night** cycle affects visibility and enemy behavior
- **Secrets** hidden in the procedural generation reward exploration

## 🎮 Core Design Principles

| Principle | Implementation |
|-----------|----------------|
| **2.5D Side-Scrolling** | Orthographic camera, parallax layers, left/right/jump movement |
| **Responsive** | Scales to any screen (phone portrait/landscape, tablet, desktop) |
| **Procedural** | Simplex noise generates infinite coherent worlds |
| **Juicy** | Visual feedback, screen shake, particle effects, haptics |
| **Accessible** | Clear UI, PICO-8 palette, simple binary controls |

## 🛠 Technology Stack

```
Strata Core → React Three Fiber, Zustand, Miniplex ECS
Strata Presets → Reusable game assets (creatures, obstacles, collectibles)
Strata Shaders → Sky, water, volumetrics, terrain rendering
Strata Capacitor → Mobile deployment (iOS/Android), haptics, storage
Three.js → 3D rendering with orthographic 2.5D view
simplex-noise → Coherent procedural world generation
```

## 📁 Project Structure

```
packages/
└── game/                    # Core game package
    └── src/
        ├── components/      # React components
        │   └── Game.tsx     # Root component with responsive camera
        ├── core/            # Engine, sprites, store
        │   ├── engine.ts    # 2D drawing primitives (port of pygame)
        │   ├── sprites.ts   # Procedural sprite drawing
        │   └── store.ts     # Zustand game state
        ├── scenes/          # Game screens (state machine)
        │   ├── Splash.tsx
        │   ├── MainMenu.tsx
        │   ├── Playing.tsx  # Main gameplay (side-scroller)
        │   └── ...
        ├── systems/         # Game systems
        │   ├── weather.ts   # Dynamic weather
        │   ├── time.ts      # Day/night cycle
        │   └── ecs.ts       # Miniplex entity system
        ├── ui/              # UI components
        │   └── VirtualControls.tsx  # BiJoystick + ActionPad triggers
        ├── world/           # World generation
        │   └── generation.ts
        └── types/           # TypeScript types
            └── index.ts

apps/
└── web/                     # Web/mobile app
    ├── src/
    │   ├── main.tsx
    │   └── App.tsx
    ├── capacitor.config.ts  # Mobile deployment config
    └── package.json
```

## 🎯 Development Commands

```bash
# Development
pnpm dev              # Run dev server
pnpm build            # Build for production
pnpm typecheck        # Type check all packages

# Mobile Deployment (from apps/web)
pnpm cap:add:ios      # Add iOS platform
pnpm cap:add:android  # Add Android platform
pnpm mobile:build     # Build and sync to native
pnpm cap:open:ios     # Open in Xcode
pnpm cap:open:android # Open in Android Studio
```

## ⚡ Key Technical Decisions

### 2.5D Side-Scrolling with R3F
```tsx
<OrthographicCamera
  left={-viewWidth / 2}
  right={viewWidth / 2}
  top={viewHeight / 2}
  bottom={-viewHeight / 2}
/>
```
The game renders with an orthographic camera looking from the side. Parallax layers are positioned at different Z-depths for visual depth.

### Responsive Scaling
```tsx
const ASPECT_RATIO = 16 / 9;
// Game scales to fit any screen while maintaining aspect ratio
// Letterboxing applied automatically
```

### Binary Trigger Controls (not analog joystick)
```tsx
// LEFT/RIGHT triggers (Simon Says style)
<TriggerButton onActivate={() => moveLeft()} />
<TriggerButton onActivate={() => moveRight()} />

// 4 action buttons (Jump, Crouch, Attack, Defend)
<ActionPad ... />
```

### Strata Integration
- **TriggerComposer pattern** for touch controls
- **Capacitor plugin** for mobile deployment with haptics
- **Device detection** for responsive layout
- **Safe area insets** for notches/rounded corners

## 🎨 Visual Identity

- **Palette**: 16-color PICO-8 inspired aesthetic
- **Resolution**: 480x270 base, scales responsively
- **Style**: Procedural pixel art, 3D rendered as 2.5D
- **Feedback**: Color flashes, screen shake, haptic vibration

## 🔊 Audio

- Procedural ambient sounds based on biome
- Weather audio (rain, wind, thunder)
- Satisfying combat sounds
- Subtle music that responds to danger level

## 📱 Controls

**Keyboard**: A/D or Arrows for move, W/Space for jump, X for attack, Z for defend
**Touch**: BiJoystick (binary left/right) + ActionPad (4 buttons)
**Gamepad**: D-pad + face buttons (via Capacitor plugin)

## 🤖 AI Constraints & Guidelines

### ✅ Required
- Use Strata patterns (TriggerComposer, not analog joystick)
- Responsive design (scale to any screen)
- Side-scrolling 2.5D perspective (orthographic camera)
- PICO-8 palette for all colors
- Procedural sprites (no external image assets)

### 🚫 Prohibited
- Full analog joysticks (use binary triggers)
- Fixed 256x256 resolution (must be responsive)
- Top-down or isometric perspective (side-scroll only)
- External image assets (.png, .jpg)
- Skipping game states (must follow state machine)

## Before Making Changes

1. Ask: "Does this enhance the mobile play experience?"
2. Check device responsiveness on different screen sizes
3. Test touch controls work intuitively
4. Verify the side-scrolling perspective is maintained
5. **Coordinate**: Rely on **Ecosystem Triage** (Synthesis Issues) for merge decisions.
6. **Communicate**: Address AI-generated feedback in PR threads to clear tracking issues.

## Coding Standards

- TypeScript strict mode
- React Three Fiber + Drei for 3D
- Zustand + Immer for state
- Strata patterns for controls and deployment
- Conventional commits (feat/fix/docs/test/chore)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arcade-cabinet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arcade-cabinet)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
