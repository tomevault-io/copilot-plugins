## big-chakra-hustle

> Vanilla JS ES6 module game. No build step, no bundler, no dependencies. Runs directly via `npx serve .` then open `http://localhost:3000`.

# BIG CHAKRA HUSTLE — Project Guide

## Architecture

Vanilla JS ES6 module game. No build step, no bundler, no dependencies. Runs directly via `npx serve .` then open `http://localhost:3000`.

### File Structure

```
index.html              Entry point
game.css                Minimal styles (fullscreen canvas)
manifest.json           PWA manifest
sw.js                   Service worker for offline caching
src/
  main.js               Bootstrap — creates Game, calls start()
  config.js             All tunable constants (physics, colors, states)
  engine/
    Game.js             Master state machine, update/draw orchestrator
    Input.js            Keyboard + touch input with action mapping
    Timer.js            Delta time, slow-motion, FPS tracking
    Camera.js           Follow camera with shake + lerp
  entities/
    Player.js           Player physics, zodiac switching, combat
    Platform.js         Platform rendering
    Shard.js            Collectible items (chakra, health, token)
    Enemy.js            Enemy AI state machine (patrol/chase/attack)
    bosses/
      BossBase.js       Multi-phase boss system
      AllBosses.js      Boss factory + per-world boss variants
  systems/
    ParticleSystem.js   Object-pooled particle effects
    LevelGenerator.js   Procedural room/platform generation
    WorldThemes.js      Per-chakra background rendering
    AudioEngine.js      Procedural music + SFX via Web Audio API
    ChakraPowers.js     7 unlockable abilities
    TokenEconomy.js     Star Token currency system
    SaveSystem.js       localStorage persistence
  data/
    ChakraData.js       Chakra definitions (colors, frequencies, names)
    astroEngine.js      Birth chart calculation
    NakshatraEffects.js Lunar mansion passive abilities
    DecanModifiers.js   Decan-based world generation modifiers
  graphics/
    DrawLib.js          Canvas drawing helpers
    SacredGeometry.js   Flower of Life, Metatron's Cube renderers
    Effects.js          Post-processing (scanlines, vignette)
  ui/
    HUD.js              Health/chakra bars, score, power wheel
    MenuSystem.js       Shop screen
    Notifications.js    Floating text notifications
    CharacterCreation.js Sun/Moon/Rising selection UI
    CoherenceMinigame.js Simon-like chakra sequence game
    DialogSystem.js     Boss dialog overlay
```

### Key Patterns

- **State machine**: `Game.state` controls flow (TITLE → CHARACTER_CREATE → PLAYING → GAME_OVER)
- **Action-based input**: `Input.actions.jump` / `Input.actionJustPressed('attack')` — all code reads actions, not raw keys
- **Zodiac duality**: Player has two physics profiles (AIRES/KOIDON) switchable at runtime
- **CONFIG-driven**: All constants in `src/config.js` — physics, colors, spawn rates, etc.
- **No external assets**: All graphics are canvas-drawn, all audio is procedurally generated

### Testing

```bash
npx serve .
# Open http://localhost:3000
```

No test framework — manual playtesting. Check browser console for errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Drocai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Drocai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
