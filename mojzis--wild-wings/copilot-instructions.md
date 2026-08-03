## wild-wings

> This file provides comprehensive guidance for AI-assisted development on the Wild Wings project.

# 🤖 CLAUDE Development Guide

This file provides comprehensive guidance for AI-assisted development on the Wild Wings project.

## 📁 Project Structure

```
wild-wings-prototype/
├── src/
│   ├── App.js                      # Root component, state routing
│   ├── components/                 # React UI components
│   │   ├── GameCanvas.jsx          # Main 60fps game loop
│   │   ├── MainMenu.jsx            # Level selection UI
│   │   ├── Settings.jsx            # Flight sensitivity controls
│   │   └── ElderEncounter.jsx      # Bird facts display modal
│   ├── game/                       # Core game logic (classes)
│   │   ├── Level.js                # Level definitions & generation
│   │   ├── Player.js               # Bird character & physics
│   │   ├── Physics.js              # Physics constants
│   │   ├── AbilitySystem.js        # Ability management
│   │   ├── Obstacle.js             # Collision objects
│   │   ├── Collectible.js          # Wind feathers
│   │   └── GameStateManager.js     # Save/load, progression
│   └── data/
│       └── birdFacts.js            # Educational content
└── public/
    └── index.html                  # Entry point
```

## 🎯 Key Architectural Principles

### 1. Separation of Concerns
- **Components (`/components`)**: React UI, rendering, user interaction
- **Game Logic (`/game`)**: Pure JavaScript classes, no React dependencies
- **Data (`/data`)**: Static content, configuration

### 2. Game Loop Architecture
- **GameCanvas.jsx** is the orchestrator (60fps loop)
- Game classes handle their own state and logic
- React state triggers re-renders only when necessary
- Canvas rendering happens outside React lifecycle

### 3. State Management
- **Local State**: React hooks for UI state
- **Game State**: Class instances in GameCanvas
- **Persistent State**: GameStateManager + localStorage
- **Global Settings**: localStorage for user preferences

## 🔑 Key Components Deep Dive

### GameCanvas.jsx (Main Game Loop)
**Location:** `src/components/GameCanvas.jsx`

**Responsibilities:**
- 60fps game loop via `requestAnimationFrame`
- Input handling (keyboard events)
- Physics updates (gravity, velocity, position)
- Collision detection (obstacles, collectibles, safe zones)
- Rendering pipeline (background → player → HUD)
- Game state transitions

**Key Methods:**
- `gameLoop()` - Main update/render cycle
- `handleInput()` - Keyboard state processing
- `render()` - Canvas drawing pipeline

**State Variables:**
- `playerRef` - Player instance
- `levelRef` - Current Level instance
- `abilitySystemRef` - Ability management
- `cameraOffsetX` - Horizontal scrolling position

### Level.js (Level Definition)
**Location:** `src/game/Level.js`

**Current Levels:**
- `Level.createLevel1()` - "First Flight" (lines 273-303)
- `Level.createLevel2()` - "Storm Chaser" (lines 305-347)

**Adding New Levels:**
```javascript
static createLevel3() {
  const config = {
    id: 3,
    name: 'Your Level Name',
    difficulty: 'medium',
    width: 3000,
    obstacles: [/* obstacle configs */],
    collectibles: [/* collectible configs */],
    safeZones: [/* safe zone configs */]
  };
  return new Level(config);
}
```

**Obstacle Generation:**
- Uses `generateRandomizedBranches()` for dynamic layouts
- Ensures minimum gaps (100px horizontal, 120px vertical)
- Avoids safe zones with 150px buffer

### Player.js (Bird Character)
**Location:** `src/game/Player.js`

**Physics Constants (from Physics.js):**
- Gravity: 0.20 (base, scales with sensitivity)
- Flap Strength: -8 (upward velocity)
- Terminal Velocity: 6
- Horizontal Speed: 3px/frame

**Key Methods:**
- `update(input, level)` - Physics calculations
- `render(ctx, cameraOffsetX)` - Bird drawing
- `getBounds()` - Collision box

**Ability Integration:**
- Player checks `abilitySystem.activeAbility`
- Modifies physics based on active ability
- Renders particle effects

### AbilitySystem.js (Ability Management)
**Location:** `src/game/AbilitySystem.js`

**Ability States:**
- `ready` - Can be activated
- `active` - Currently in effect
- `cooldown` - Recovering, cannot activate

**Adding New Abilities:**
1. Add ability config to `this.abilities` map
2. Update `birdFacts.js` with educational content
3. Implement effect in `Player.update()` or `GameCanvas.gameLoop()`

**Example:**
```javascript
abilities: {
  'your_ability': {
    id: 'your_ability',
    name: 'Ability Name',
    duration: 3000,    // ms
    cooldown: 10000,   // ms
    state: 'locked',
    lastUsed: 0
  }
}
```

### birdFacts.js (Educational Content)
**Location:** `src/data/birdFacts.js`

**Structure:**
```javascript
{
  id: number,
  species: string,
  fact: string,           // 50-100 words, Lexile 200-500L
  ability: string,        // Ability ID
  abilityName: string,
  color: string,          // Hex color
  icon: string           // Emoji
}
```

**Content Guidelines:**
- Keep facts age-appropriate (grades 1-3)
- Include specific numbers/measurements
- Connect fact to ability unlock
- Use engaging, enthusiastic tone

## 🧪 Testing Strategy

### Current Tests
**Location:** `src/components/GameCanvas.test.jsx`

**Coverage:**
- Component rendering
- Input handling
- Game state initialization

### Testing Conventions
- Use React Testing Library
- Mock canvas context (`getContext`)
- Test user interactions, not implementation details
- Run: `npm test`

### Adding Tests
```javascript
import { render, screen } from '@testing-library/react';
import ComponentName from './ComponentName';

test('description of what it does', () => {
  render(<ComponentName />);
  // assertions
});
```

## 🎨 Visual Design Patterns

### Color Palette
**Defined in:** Component files (consider extracting to theme)

- Sky: `#E8D5E8` → `#F5E6D3` (gradient)
- Player: `#8BA3B8` (dusty blue)
- Obstacles: `#B09A8A` (soft taupe)
- Collectibles: `#F4C430`, `#FFD700` (gold)
- Safe Zones: `#C8E6D0` (mint green)

### Canvas Rendering Order
1. Background gradient
2. Parallax clouds (15 layers)
3. Ground
4. Safe zones (inactive)
5. Obstacles
6. Collectibles
7. Player
8. HUD overlay

### Animation Patterns
- **Collectibles:** Sine wave float (`Math.sin(Date.now() / 1000)`)
- **Player:** Wing flap offset (-10px on flap)
- **Particles:** Expanding circles with fade-out

## 🔧 Common Development Tasks

### Adding a New Level
1. Open `src/game/Level.js`
2. Create static method: `Level.createLevelN()`
3. Define config object with obstacles, collectibles, safe zones
4. Update `src/components/MainMenu.jsx` to display new level
5. Test: Ensure previous level completion unlocks new level

### Adding a New Bird Species
1. Open `src/data/birdFacts.js`
2. Add new object with id, species, fact, ability, color, icon
3. Create or link ability in `src/game/AbilitySystem.js`
4. Implement ability effect in `src/game/Player.js` or `GameCanvas.jsx`
5. Test: Verify safe zone encounter displays correctly

### Adjusting Physics
1. Open `src/game/Physics.js`
2. Modify constants (gravity, flapStrength, terminalVelocity)
3. Test with different sensitivity settings
4. Consider impact on existing levels

### Modifying Obstacle Generation
1. Open `src/game/Level.js`
2. Edit `generateRandomizedBranches()` method
3. Adjust min/max gaps, counts, positioning
4. Test for impossible passages

## 🐛 Common Issues & Solutions

### Issue: Obstacle Too Dense
**Solution:** Increase `minHorizontalGap` or `minVerticalGap` in `generateRandomizedBranches()`

### Issue: Physics Feel Wrong
**Solution:** Adjust sensitivity multiplier in Settings, or modify base constants in `Physics.js`

### Issue: Abilities Not Unlocking
**Solution:** Check `GameStateManager.unlockAbility()` is called after safe zone encounter

### Issue: localStorage Not Saving
**Solution:** Verify `GameStateManager.saveProgress()` is called on level completion/ability unlock

### Issue: Collision Detection False Positives
**Solution:** Review AABB bounds calculation in `Player.getBounds()` and `Obstacle.checkCollision()`

## 📐 Code Style Guidelines

### JavaScript
- Use ES6+ features (classes, arrow functions, const/let)
- Prefer functional methods (map, filter, reduce)
- Keep functions small and focused

### React
- Functional components with hooks
- Use `useRef` for game instances (avoid re-renders)
- Keep JSX readable with proper indentation

### Canvas
- Always save/restore context: `ctx.save()` ... `ctx.restore()`
- Use meaningful variable names for coordinates
- Comment complex math/physics calculations

### Game Logic
- Classes for entities (Player, Obstacle, Collectible)
- Static methods for level creation
- Pure functions for physics calculations

## 🔍 Debugging Tips

### Visual Debugging
- Add `console.log()` in game loop (use sparingly, 60fps!)
- Draw collision bounds: `ctx.strokeRect(x, y, width, height)`
- Show player velocity on HUD

### State Debugging
- Inspect localStorage: `localStorage.getItem('wildWingsGameState')`
- Log ability state changes in `AbilitySystem.update()`
- Track level completion in `GameCanvas.gameLoop()`

### Performance Debugging
- Use Chrome DevTools Performance tab
- Monitor frame rate: `console.log(1000 / deltaTime)`
- Check for memory leaks in game loop

## 📚 Important Files Reference

### Game Logic
- **Level definitions:** `src/game/Level.js:273-347`
- **Player physics:** `src/game/Player.js:43-102` (update method)
- **Ability effects:** `src/game/Player.js:60-80` (ability checks)
- **Collision detection:** `src/components/GameCanvas.jsx:250-290`

### UI Components
- **Level selection:** `src/components/MainMenu.jsx:50-120`
- **Settings panel:** `src/components/Settings.jsx:30-80`
- **Bird encounters:** `src/components/ElderEncounter.jsx:20-100`

### Data & Config
- **Bird facts:** `src/data/birdFacts.js:1-50`
- **Physics constants:** `src/game/Physics.js:1-20`

## 🚀 Performance Considerations

### Canvas Optimization
- Clear only necessary regions (not full canvas each frame)
- Use `requestAnimationFrame` for consistent 60fps
- Batch render calls when possible

### Collision Optimization
- Check broad phase before narrow phase
- Skip off-screen objects
- Cache bounds calculations

### State Updates
- Minimize React re-renders (use refs for game state)
- Debounce expensive calculations
- Use localStorage efficiently (don't save every frame)

## 🎓 Educational Content Guidelines

### Writing Bird Facts
- **Length:** 50-100 words
- **Reading Level:** Lexile 200-500L (grades 1-3)
- **Structure:** Hook sentence → details → connection to ability
- **Vocabulary:** Mix Tier 1 (common), Tier 2 (academic), Tier 3 (domain-specific)

### Ability-Fact Alignment
- Ability should reflect real bird characteristic
- Make connection explicit in fact text
- Use specific measurements (speed, distance, time)

## 🔄 Git Workflow

- **Main Branch:** Protected, production-ready code
- **Feature Branches:** `claude/feature-name-{session-id}`
- **Commit Style:** Descriptive, present tense ("Add level 3 with new obstacles")
- **Push Command:** `git push -u origin <branch-name>`

## 📋 Checklist for New Features

- [ ] Code written and tested locally
- [ ] No console errors or warnings
- [ ] Existing levels still work
- [ ] Progress saves/loads correctly
- [ ] Physics feel consistent
- [ ] Educational content proofread
- [ ] Comments added for complex logic
- [ ] Committed with clear message

---

**Quick Reference:**
- Start development: `npm start`
- Run tests: `npm test`
- Build: `npm run build`
- Current levels: 2 (Level.js:273-347)
- Current abilities: 3 (birdFacts.js)
- Physics constants: Physics.js:1-20

**For Questions:**
- Check this file first
- Review related source code
- Test in browser dev environment
- Document new patterns discovered

---

*This guide is maintained for AI development assistants. Update as the project evolves.*

---
> Source: [mojzis/wild-wings](https://github.com/mojzis/wild-wings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
