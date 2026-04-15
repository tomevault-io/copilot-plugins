## fishbowl

> A cooperative word-guessing party game based on Fishbowl/Monikers.

# Triple Take - Project Documentation

A cooperative word-guessing party game based on Fishbowl/Monikers.

## Project Overview

Triple Take is a web-based party game where players guess words through three rounds with increasingly restrictive clues. The same words are used in all rounds, creating memorable moments as restrictions increase.

**Game Name:** Triple Take (repository may still be called "fishbowl")

### Game Mechanics

**Core Rounds:**
1. **Round 1 - Describe It**: Use any words or gestures (except the word itself)
2. **Round 2 - One Word**: Give only ONE word as a clue (can repeat it)
3. **Round 3 - Charades**: Act it out, no words allowed (sound effects OK)

**Optional Bonus Rounds:**
4. **Round 4 - Under the Bedsheet**: Act it out under a bedsheet
5. **Round 5 - Sound Effects Only**: Only sound effects, no words or acting

**Gameplay Flow:**
- Each player gets a turn (default 60 seconds, configurable: 30s-120s)
- Players pass a single device between turns (in-person mode)
- Round ends when ALL words have been guessed (may take many turns)
- Game continues through all selected rounds
- Cooperative scoring: track total words guessed, not competitive

**Key Features:**
- Skip words (they shuffle back into the deck randomly)
- Pause/resume functionality
- Turn-by-turn statistics
- Round-by-round breakdown
- Auto-save to LocalStorage

---

## Tech Stack

### Core Technologies
- **Vite** - Build tool and dev server
- **TypeScript** - Type-safe development (strict mode enabled)
- **Vanilla JavaScript** - No framework (React/Vue/etc)
- **CSS with Custom Properties** - For theming and responsiveness
- **LocalStorage** - Game state persistence

### Design Principles
- **Jamstack approach**: Build upfront, serve from CDN
- **Mobile-first**: Responsive design optimized for phones
- **No backend required**: Fully client-side (Phase 1)
- **Minimal dependencies**: Keep bundle size small
- **Progressive enhancement**: Works without installation, PWA-ready

### Deployment
- **Platform**: Netlify (or any static host)
- **Build command**: `npm run build`
- **Publish directory**: `dist`
- **Zero cost**: No server, no database, no API calls

---

## Project Structure

```
fishbowl/
├── public/
│   └── wordlists/          # Default word lists (plain text)
│       ├── classic.txt     # 50 classic words
│       ├── pop-culture.txt # 50 modern references
│       └── mixed.txt       # 60 varied difficulty
├── src/
│   ├── main.ts             # Application entry point
│   ├── types.ts            # All TypeScript interfaces
│   ├── constants.ts        # Game constants and round definitions
│   ├── state/
│   │   ├── GameState.ts    # Centralized state manager
│   │   └── storage.ts      # LocalStorage utilities
│   ├── screens/
│   │   ├── BaseScreen.ts         # Base class for all screens
│   │   ├── ScreenManager.ts      # Navigation system
│   │   ├── WelcomeScreen.ts      # Entry point
│   │   ├── SetupScreen.ts        # Game configuration
│   │   ├── WordInputScreen.ts    # Choose/paste/upload words
│   │   ├── ReviewScreen.ts       # Verify word list
│   │   ├── RoundIntroScreen.ts   # Round rules explanation
│   │   ├── TurnIntroScreen.ts    # Pass device prompt
│   │   ├── PlayScreen.ts         # Core gameplay
│   │   ├── TurnEndScreen.ts      # Turn statistics
│   │   ├── RoundEndScreen.ts     # Round statistics
│   │   └── GameEndScreen.ts      # Final results
│   ├── components/
│   │   └── Timer.ts        # Countdown timer with pause/resume
│   ├── utils/
│   │   ├── shuffle.ts      # Fisher-Yates algorithm
│   │   └── wordLoader.ts   # Load words from various sources
│   └── styles/
│       ├── variables.css   # Design tokens (colors, spacing, etc)
│       └── main.css        # Base styles and components
├── index.html              # HTML entry point
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript configuration (strict mode)
├── vite.config.ts          # Vite build configuration
├── .gitignore              # Excludes Monikers+PnP.pdf reference
└── README.md               # Project documentation
```

---

## Architecture & Design Patterns

### State Management

**Centralized GameStateManager** (singleton pattern):
- Single source of truth for all game state
- Subscriber pattern for reactive UI updates
- Immutable state updates (copy-on-write)
- Auto-save to LocalStorage with debouncing (max once/second)

**Key State Methods:**
- `startNewGame(config, words)` - Initialize game
- `startTurn()` - Begin a new turn
- `markWordCorrect()` - Mark current word as guessed
- `skipWord()` - Shuffle word back into deck
- `updateTimer(remaining)` - Update countdown
- `pauseTurn()` / `resumeTurn()` - Pause functionality
- `endTurn()` - Record turn stats, increment turn number
- `completeRound()` - Finalize round, prepare next round or end game

### Screen Navigation

**ScreenManager** handles routing:
- Maps screen IDs to screen factory functions
- Supports both sync and async render methods
- Clears and re-renders on navigation

**Screen Flow:**
```
Welcome
  ↓
Setup (configure timer, select rounds)
  ↓
Word Input (built-in lists / paste / upload)
  ↓
Review (verify and edit word list)
  ↓
Round Intro (explain round rules)
  ↓
Turn Intro (pass device prompt) ←─┐
  ↓                                │
Play (60s timer, word display)     │
  ↓                                │
Turn End (turn stats) ─────────────┘ (repeat until all words guessed)
  ↓
Round End (round stats)
  ↓
[Next Round Intro or Game End]
  ↓
Game End (final results)
```

### Critical Implementation Details

#### Round Completion Logic

**Problem:** When the last word is guessed, multiple state updates happen:
1. `markWordCorrect()` removes word from `remainingWords`
2. `endTurn()` records turn stats
3. `completeRound()` refills words for next round

**Solution:** Check BEFORE calling `markWordCorrect()` if this is the last word:
```typescript
const willCompleteRound = stateBefore.remainingWords.length === 1;
getGameState().markWordCorrect();
if (willCompleteRound) {
  navigate('round-end');
}
```

#### Turn History Management

**Problem:** `endTurn()` needs to append to current round's history, not overwrite previous rounds.

**Solution:** Check if the last round history entry is for the current round:
```typescript
const isNewRound = !lastRoundHistory ||
  lastRoundHistory.roundNumber !== state.currentRound;

if (isNewRound) {
  // Append new round history entry
} else {
  // Update existing round entry with new turn
}
```

#### Skip Word Implementation

Words are shuffled back into a RANDOM position (not at the end):
```typescript
// Remove current word
const newRemaining = [...state.remainingWords];
newRemaining.splice(currentWordIndex, 1);

// Insert at random position
const randomIndex = Math.floor(Math.random() * (newRemaining.length + 1));
newRemaining.splice(randomIndex, 0, currentWord);
```

This prevents word clustering and feels more natural.

---

## Design System

### Color Palette

**Light Mode:**
- Primary: `#6366f1` (Indigo)
- Secondary: `#ec4899` (Pink)
- Success: `#10b981` (Green) - "Got It!" button
- Warning: `#f59e0b` (Amber) - "Skip" button
- Error: `#ef4444` (Red)
- Background: `#ffffff`
- Surface: `#f9fafb`
- Text: `#111827`

**Dark Mode:** Automatically activated via `prefers-color-scheme: dark`

### Typography
- Font: System font stack (system-ui, -apple-system, sans-serif)
- Sizes: sm(0.875rem), base(1rem), lg(1.25rem), xl(1.5rem), 2xl(2rem), 3xl(3rem)

### Spacing
- xs: 0.5rem, sm: 1rem, md: 1.5rem, lg: 2rem, xl: 3rem

### Touch Targets
- Minimum 44x44px for all interactive elements
- Large "Got It!" button for easy tapping during gameplay

---

## Development Workflow

### Commands

```bash
# Install dependencies
npm install

# Start dev server (http://localhost:5173)
npm run dev

# Build for production (outputs to dist/)
npm run build

# Preview production build
npm run preview
```

### TypeScript Configuration

Strict mode enabled with:
- `strict: true`
- `noUnusedLocals: true`
- `noUnusedParameters: true`
- `noFallthroughCasesInSwitch: true`
- `noUncheckedIndexedAccess: true`

### Git Workflow

Repository is on GitHub. Commits follow conventional format:
```
Brief description of change

- Detailed bullet points
- Of what was changed
- And why

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

---

## Known Issues & Solutions

### Issue: Timer precision
**Problem:** `setInterval` can drift over time
**Current:** Acceptable for party game use
**Future:** Could recalculate based on elapsed time for precision

### Issue: Multiple tabs
**Problem:** Opening game in two tabs causes state conflicts
**Current:** Not handled
**Future:** Use storage event listeners to sync or warn

### Issue: Browser back button
**Problem:** Not integrated with navigation
**Current:** Ignored
**Future:** Could use History API

---

## Future Enhancements

### Phase 2: Online Multiplayer
- Use Supabase for real-time sync
- Rooms with join codes
- Multiple devices, one game
- Netlify integration available for Supabase

### Additional Features
- **LLM Word Generation**: Users provide their own API key (OpenAI/Anthropic)
- **PWA Manifest**: Install as app on phone
- **Sound Effects**: Audio feedback for actions
- **Confetti Animation**: On game completion
- **Export Stats**: Download game results
- **Custom Round Order**: Let users rearrange rounds
- **More Word Lists**: Expand default content

---

## Word List Format

Plain text files, one word per line:
```
# Comments start with #
Abraham Lincoln
Beyoncé
Harry Potter
Pizza
```

**Creating Custom Lists:**
- Save as `.txt` file
- One word/phrase per line
- Lines starting with `#` are ignored
- Upload via Word Input screen

---

## Deployment Notes

### Netlify Setup
1. Connect GitHub repository
2. Build command: `npm run build`
3. Publish directory: `dist`
4. Build settings: Node 18+
5. No environment variables needed

### Other Platforms
Works on any static host:
- Vercel
- GitHub Pages
- Cloudflare Pages
- AWS S3 + CloudFront
- Any CDN

Just serve the `dist/` folder after running `npm run build`.

---

## Code Style & Conventions

### TypeScript
- Use strict types always
- Prefer `interface` over `type` for objects
- Use `readonly` for immutable state
- Explicit return types on public methods

### State Updates
- Always create new objects (immutability)
- Never mutate state directly
- Use spread operator for copies: `{...state, newProp: value}`

### Screen Classes
- Extend `BaseScreen`
- Implement `render(): HTMLElement | Promise<HTMLElement>`
- Use helper methods: `createButton()`, `createHeader()`, etc.
- Clean up listeners in `cleanup()` method if needed

### Naming Conventions
- PascalCase for classes and types
- camelCase for variables and functions
- UPPER_SNAKE_CASE for constants
- Descriptive names over abbreviations

---

## Testing Checklist

### Setup Flow
- [ ] Configure timer options (test each duration)
- [ ] Select different round combinations
- [ ] Load built-in word lists
- [ ] Paste custom words
- [ ] Upload text file
- [ ] Review and remove words

### Gameplay
- [ ] Timer counts down correctly
- [ ] Timer changes color (warning at 30s, danger at 10s)
- [ ] "Got It!" button marks word correct
- [ ] "Skip" button shuffles word back
- [ ] Pause overlay works
- [ ] Resume continues from correct time
- [ ] Turn ends when timer reaches 0
- [ ] Turn End screen shows correct stats

### Round Completion
- [ ] Round ends when all words guessed
- [ ] Round End screen shows correct total
- [ ] Next round has all words back
- [ ] Words are properly shuffled

### Multi-Round Game
- [ ] Play through 2+ rounds
- [ ] Round history preserved
- [ ] Game End shows all rounds
- [ ] Statistics are accurate

### Edge Cases
- [ ] Guess last word before timer expires
- [ ] Skip all words multiple times
- [ ] Pause/resume repeatedly
- [ ] Close tab mid-game, then return (should resume)
- [ ] Play Again clears state properly

### Mobile Testing
- [ ] All buttons are tappable
- [ ] Text is readable
- [ ] Layout works in portrait
- [ ] Layout works in landscape
- [ ] Timer is visible while playing

---

## Troubleshooting

### Build Fails
1. Check TypeScript errors: `npm run build`
2. Ensure no unused variables/imports
3. Verify all files compile with strict mode

### LocalStorage Not Saving
1. Check browser storage quota
2. Verify no privacy mode blocking storage
3. Check console for errors

### Timer Issues
1. Verify Timer component is cleaned up on navigation
2. Check for multiple timers running simultaneously
3. Ensure state updates call `updateTimer()`

### Navigation Issues
1. Check ScreenManager has all screens registered
2. Verify screen factory functions are correct
3. Check for navigation loops

---

## Resources & References

### Game Origins
- **Fishbowl**: Traditional party game (public domain)
- **Monikers**: Commercial card game by Alex Hague and Justin Vickers
  - Licensed under Creative Commons BY-NC-SA 4.0
  - Cannot use "Monikers" name commercially
  - Can adapt game mechanics for personal/non-commercial use

### Similar Projects
- https://fishbowl-game.com/ - Open source online version
- https://github.com/avimoondra/fishbowl - Reference implementation

### Technologies
- Vite: https://vitejs.dev/
- TypeScript: https://www.typescriptlang.org/
- Netlify: https://docs.netlify.com/

---

## Contact & Contribution

This is a personal project. If you'd like to contribute or have questions:
- Open an issue on GitHub
- Follow the existing code style
- Test thoroughly before submitting PRs
- Keep the game free and accessible

---

## License

This project is open source. The game mechanics are based on the traditional party game Fishbowl (public domain) and inspired by Monikers (CC BY-NC-SA 4.0).

The code in this repository is available for personal and non-commercial use. If you adapt it, please provide attribution and share under the same terms.

**Cannot:**
- Use commercially without permission
- Use the name "Triple Take" commercially (choose your own name!)
- Remove attribution

**Can:**
- Use for personal parties
- Modify and adapt
- Deploy your own instance
- Learn from the code

---

## Quick Start for New Developers

1. **Clone and setup:**
   ```bash
   git clone <repo-url>
   cd fishbowl  # (or triple-take if renamed)
   npm install
   npm run dev
   ```

2. **Read this document** (claude.md) fully

3. **Check the plan:** `.claude/plans/deep-cooking-corbato.md` has the original detailed implementation plan

4. **Key files to understand:**
   - `src/types.ts` - Understand the data structures
   - `src/state/GameState.ts` - Core game logic
   - `src/screens/PlayScreen.ts` - Most complex UI
   - `src/constants.ts` - Game configuration

5. **Make changes:**
   - Test locally with `npm run dev`
   - Build with `npm run build`
   - Verify TypeScript compiles without errors

6. **Deploy:**
   - Push to GitHub
   - Netlify auto-deploys from main branch

---

*Last updated: January 2026*
*This document should be kept up-to-date as the project evolves.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/verythorough)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/verythorough)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
