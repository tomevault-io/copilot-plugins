## retrotoolbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Static Socket Configuration

**CRITICAL**: RetroToolbox MUST use port 3004 exclusively:
- **Tailscale Access**: http://100.x.x.x:3004 (replace x.x.x with your Tailscale IP)
- **NEVER use `serve` command or dynamic ports**
- **Always use `npm run dev` which runs on port 3004**

## Project Overview

This is a React-based retro arcade game collection featuring multiple classic games with modern enhancements. The application uses a fully modular architecture with individual game components and optimized core systems.

## Development Commands

**Essential Commands for Codex:**
- `npm run dev` - Start development server on port 3004 (REQUIRED for testing)
- `npm run build` - Build production bundle
- `npm run lint` - Run ESLint for code quality checks
- `npm run lint:fix` - Auto-fix ESLint issues
- `npm run typecheck` - TypeScript type checking without building
- `npm install` - Install dependencies (run after package.json changes)

## Architecture

**✅ Fully Modular Implementation Complete**

The application uses a complete modular React architecture with extracted components:
- **Main App**: `src/App.tsx` handles game routing, settings, and statistics
- **Game Components**: Individual games in `src/components/games/` with full implementations
- **Core Systems**: Optimized shared systems in `src/core/` for audio, input, and particles
- **UI Components**: Reusable UI elements in `src/components/ui/` for menus and effects

**Extracted Core Systems**

All core systems have been optimized and extracted to `src/core/`:
- **SoundManager.ts**: Web Audio API sound engine with programmatic effects
- **InputManager.ts**: Unified input handling for keyboard, mouse, touch, and gamepad
- **ParticleSystem.ts**: High-performance visual effects system
- **GameTypes.ts**: TypeScript interfaces and type definitions

## Key Technical Details

### State Management
- Uses React hooks (useState, useEffect, useRef, useCallback, useMemo)
- Game state persisted to localStorage (settings and high scores)
- Game loop implemented with requestAnimationFrame

### Canvas Rendering
- Each game uses HTML5 Canvas for rendering
- Responsive canvas sizing with window resize handlers
- Custom particle effects and visual feedback

### Implemented Games

**✅ All Games Fully Implemented in Modular Components:**
1. **Snake++** (`SnakeGame.jsx`): Complete with power-ups, lives system, and particle effects
2. **Neon Pong** (`PongGame.jsx`): Complete AI opponent with difficulty scaling and visual effects  
3. **Brick Breaker** (`BreakoutGame.jsx`): Complete breakout clone with multi-hit bricks and power-ups
4. **Tetris Remix** (`TetrisGame.jsx`): Complete with enhanced mechanics and animations
5. **Space Defense** (`SpaceInvadersGame.jsx`): Complete with progressive enemy waves and upgrades
6. **Pac-Man** (`PacManGame.jsx`): Complete with maze navigation and mobile touch controls

## Development Notes

### Current Development Status

**✅ Modular Architecture Complete**
1. All games successfully extracted to individual components
2. Core systems optimized and moved to `src/core/`
3. All games fully functional with enhanced features
4. Shared utilities implemented for common game functionality

**Enhancement Opportunities**
1. Add multiplayer functionality to existing games
2. Implement achievement and progression systems
3. Add new game variants or difficulty modes
4. Enhance mobile touch controls and responsiveness
5. Add background music and enhanced sound effects

### Adding New Games (Current Process)
With the modular architecture in place:
1. Create new game component in `src/components/games/`
2. Follow the established pattern from existing game components
3. Import and add to `gameComponents` object in `src/App.tsx`
4. Add game metadata to `games` array in `src/App.tsx`
5. Integrate with core systems (SoundManager, ParticleSystem, InputManager)
6. Test thoroughly on desktop and mobile devices

### Sound System
- Sound effects are generated programmatically using Web Audio API
- Methods: `playTone()`, `playCollect()`, `playHit()`, `playGameOver()`, `playPowerUp()`
- Volume and enabled state controlled through settings

### Performance Considerations
- Particle systems are pruned when life <= 0
- Canvas operations are optimized with minimal redraws
- Game loops use deltaTime for consistent gameplay across frame rates

## Testing and Debugging with Playwright MCP

### Setup Local Playwright MCP Server

**IMPORTANT**: Always use the local headless Playwright MCP server for testing changes in RetroToolbox.

#### Installation (if not already installed):
```bash
# Install as dev dependency
npm install @playwright/mcp@latest --save-dev

# Add local MCP server
claude mcp add playwright-headless -s local -- npx @playwright/mcp@latest --headless
```

#### Verify Installation:
```bash
claude mcp get playwright-headless
```

### Testing Protocol

**CRITICAL**: After EVERY code change, especially for games, you MUST:

1. **Navigate to the game**:
   ```typescript
   mcp__playwright-headless__browser_navigate({ url: "http://localhost:3004" })
   ```

2. **Take screenshots before/after changes**:
   ```typescript
   mcp__playwright-headless__browser_take_screenshot({ filename: "before-change.png" })
   // Make code changes
   mcp__playwright-headless__browser_take_screenshot({ filename: "after-change.png" })
   ```

3. **Test user interactions**:
   ```typescript
   // Click on game
   mcp__playwright-headless__browser_click({ element: "game button", ref: "element_ref" })
   
   // Test keyboard input
   mcp__playwright-headless__browser_press_key({ key: "ArrowRight" })
   
   // Check console for errors
   mcp__playwright-headless__browser_console_messages()
   ```

4. **Verify game states**:
   - Game starts properly when arrow key pressed
   - Score updates correctly
   - Game over triggers appropriately
   - Touch controls work on mobile view

### Common Testing Scenarios

#### Testing Game Start:
```typescript
// 1. Navigate to main menu
mcp__playwright-headless__browser_navigate({ url: "http://localhost:3004" })

// 2. Click on specific game
mcp__playwright-headless__browser_snapshot() // Get element refs
mcp__playwright-headless__browser_click({ element: "PAC-MAN NEON", ref: "e87" })

// 3. Press arrow key to start
mcp__playwright-headless__browser_press_key({ key: "ArrowRight" })

// 4. Verify game started (no longer shows "READY?")
mcp__playwright-headless__browser_snapshot()
```

#### Testing Mobile Touch Controls:
```typescript
// Resize to mobile dimensions
mcp__playwright-headless__browser_resize({ width: 375, height: 667 })

// Test touch interactions (swipe simulation)
// Note: Use click at different positions to simulate swipe
```

#### Debugging Issues:
```typescript
// Always check console for errors
mcp__playwright-headless__browser_console_messages()

// Take screenshot to see visual state
mcp__playwright-headless__browser_take_screenshot({ filename: "debug-state.png" })

// Get page snapshot to understand DOM structure
mcp__playwright-headless__browser_snapshot()
```

### Testing Checklist

Before considering any game change complete:

- [ ] Game loads without console errors
- [ ] Arrow keys/WASD start the game from ready state
- [ ] Game mechanics work as expected (movement, collision, scoring)
- [ ] Pause/Resume functionality works
- [ ] Game over state triggers correctly
- [ ] High score updates properly
- [ ] Sound effects play (if enabled)
- [ ] Mobile touch controls functional
- [ ] No memory leaks (check console after playing)
- [ ] Performance is smooth (no lag or stuttering)

## Advanced Debugging Methodology

### Parallel Task Investigation Framework

For complex runtime errors (like undefined property access), use the parallel debugging approach:

**When to Use:**
- Persistent runtime errors that occur in multiple places
- "Cannot read properties of undefined" errors
- Complex codebases where line numbers become unreliable after edits

**5-Track Parallel Investigation:**

1. **Track A: Property Access Patterns**
   ```bash
   # Search for direct .x access without null checks
   search: \.x[\s\)\]\;\,]
   search: \.position\.x
   search: \.velocity\.x
   ```

2. **Track B: Array Operations**
   ```bash
   # Find array operations that could create undefined elements
   search: \.splice\(
   search: \.push\(.*x:
   search: \.shift\(
   ```

3. **Track C: Event Coordinates**
   ```bash
   # Check event handling for undefined access
   search: clientX
   search: e\.touches.*\.x
   search: e\..*\.x
   ```

4. **Track D: Object Creation**
   ```bash
   # Find object initialization issues
   search: = \{.*x:
   search: create.*\(
   search: new.*\(
   ```

5. **Track E: Unprotected Loops**
   ```bash
   # Find loops without null checks
   search: forEach\(.*=>
   search: for.*of.*
   search: \.map\(
   ```

**Methodology:**
1. Launch 5 concurrent Task agents with specific search patterns
2. Each task reports HIGH/MEDIUM/LOW risk findings with context
3. Apply fixes in priority order (HIGH first)
4. Test with Playwright after each batch of fixes
5. Use pattern-based searches instead of line numbers

**Critical Success Factors:**
- **Pattern-based debugging** vs line-number dependent searches
- **Comprehensive null checking** for nested object properties
- **Real-time browser testing** with Playwright to verify fixes
- **Systematic documentation** of all fixes applied

**Example Pattern Fix:**
```typescript
// Before (HIGH RISK):
particle.velocity.x += particle.acceleration.x * deltaTime;

// After (SAFE):
if (!particle || !particle.velocity || !particle.acceleration) continue;
particle.velocity.x += particle.acceleration.x * deltaTime;
```

### Phase 4 Debugging Results

**Successfully Fixed Issues:**
- ✅ platforms[0] undefined access with empty array guard
- ✅ Enhanced camera system undefined access validation
- ✅ Particle system comprehensive null checks for velocity/acceleration/wind
- ✅ Platform cracks forEach loop with x/y validation
- ✅ Touch event handling with proper touches[0] validation
- ✅ Manager references protected with null checks
- ✅ Trail array management with proper safety checks

### Example Test Flow for Bug Fixes

```typescript
// 1. Reproduce the bug
mcp__playwright-headless__browser_navigate({ url: "http://localhost:3004" })
mcp__playwright-headless__browser_click({ element: "problematic game", ref: "ref" })
mcp__playwright-headless__browser_take_screenshot({ filename: "bug-reproduction.png" })

// 2. Check console for errors
const messages = mcp__playwright-headless__browser_console_messages()

// 3. Apply fix
// ... make code changes ...

// 4. Test fix works
mcp__playwright-headless__browser_navigate({ url: "http://localhost:3004" })
mcp__playwright-headless__browser_click({ element: "fixed game", ref: "ref" })
mcp__playwright-headless__browser_press_key({ key: "ArrowRight" })
mcp__playwright-headless__browser_take_screenshot({ filename: "bug-fixed.png" })

// 5. Verify no new errors
mcp__playwright-headless__browser_console_messages()
```

### Important Notes

1. **Always run `npm run dev` in a separate terminal** before testing
2. **Refresh the page** after making changes to see updates
3. **Check console messages** for any errors or warnings
4. **Take screenshots** to document issues and fixes
5. **Test on different screen sizes** for responsive design
6. **Verify all game states** (ready, playing, paused, game over)

Remember: Visual testing with Playwright is essential for games since many issues are visual in nature and won't show up in unit tests.

## Git Workflow and Version Control

### Committing Changes

**CRITICAL**: After completing any major upgrade, bug fix, or feature implementation, you MUST commit and push changes to GitHub.

#### Standard Git Workflow:

1. **Check current status**:
   ```bash
   git status
   git diff  # Review changes before committing
   ```

2. **Stage and commit changes**:
   ```bash
   # Stage all changes
   git add .
   
   # Or stage specific files
   git add src/components/games/PacManGame.tsx
   git add CLAUDE.md
   
   # Commit with descriptive message
   git commit -m "fix: resolve PacMan arrow key input issue
   
   - Fixed phaseTimer reference error preventing game start
   - Updated game state transition logic
   - Added comprehensive Playwright testing documentation
   
   🤖 Generated with Claude Code
   
   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

3. **Push to GitHub**:
   ```bash
   # Push to current branch
   git push
   
   # Or push to specific branch
   git push origin main
   ```

### When to Commit

**Always commit after**:
- ✅ Completing a bug fix that's been tested
- ✅ Implementing a new feature
- ✅ Major refactoring or optimization
- ✅ Updating documentation (like this CLAUDE.md file)
- ✅ Fixing critical issues that affect gameplay
- ✅ Adding new games or game mechanics
- ✅ Performance improvements
- ✅ Mobile responsiveness fixes

### Commit Message Format

Follow conventional commits format:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting, etc)
- `refactor:` Code refactoring
- `perf:` Performance improvements
- `test:` Test additions or changes
- `chore:` Build process or auxiliary tool changes

Example:
```bash
git commit -m "feat: add power-up system to PacMan game

- Implemented speed boost and shield power-ups
- Added visual effects for active power-ups
- Integrated with particle system for collection effects
- Updated scoring system for power-up bonuses

🤖 Generated with Claude Code

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Important Git Commands

```bash
# View recent commits
git log --oneline -10

# Check remote repository
git remote -v

# Pull latest changes before starting work
git pull

# Create a new branch for major features
git checkout -b feature/multiplayer-mode

# Switch between branches
git checkout main
git checkout feature/multiplayer-mode

# Merge feature branch to main
git checkout main
git merge feature/multiplayer-mode
```

### Best Practices

1. **Commit frequently**: Don't wait until everything is perfect
2. **Test before committing**: Use Playwright to verify changes work
3. **Write clear commit messages**: Future developers (including you) will thank you
4. **Pull before pushing**: Avoid merge conflicts
5. **Never commit broken code**: Always test first
6. **Include the AI attribution**: Add the Claude Code signature to commits

### After Major Updates Checklist

- [ ] All tests pass (manual testing with Playwright)
- [ ] No console errors
- [ ] Code is properly formatted
- [ ] Documentation is updated if needed
- [ ] Changes are committed with descriptive message
- [ ] Changes are pushed to GitHub
- [ ] Verify push succeeded: `git log origin/main`

### Example Complete Workflow

```bash
# 1. Make changes to fix PacMan input issue
# ... code changes ...

# 2. Test with Playwright
# ... run Playwright tests ...

# 3. Check what changed
git status
git diff

# 4. Stage and commit
git add -A
git commit -m "fix: resolve PacMan game input and rendering issues

- Fixed missing phaseTimer causing arrow keys to not start game
- Corrected game state transitions
- Added comprehensive testing documentation
- Improved error handling in stats validation

🤖 Generated with Claude Code

Co-Authored-By: Claude <noreply@anthropic.com>"

# 5. Push to GitHub
git push

# 6. Verify push succeeded
git log --oneline -1 origin/main
```

Remember: Version control is crucial for tracking progress, enabling collaboration, and allowing rollback if issues arise.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ildunari) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
