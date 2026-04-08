## kachuful

> > **Project:** Kachuful Card Game Scoring Application

# Kachuful Scorer - Antigravity Development Guide

> **Project:** Kachuful Card Game Scoring Application  
> **AI Agent:** Antigravity by Google Deepmind  
> **Purpose:** Step-by-step guide for building the app using Antigravity

---

## 🎯 Project Overview

This application tracks scores for **Kachuful** (also called Judgement), a trick-taking card game popular in India. The app automates score calculation, manages rounds, and provides statistics.

**Key Game Mechanics:**
- 2-10 players predict exact number of tricks they'll win
- Scoring: `10 + predicted tricks` if accurate, `0` if wrong
- Dealer restriction: Last bidder can't make total bids equal total tricks
- Trump rotates: Spades → Diamonds → Clubs → Hearts → No Trump

---

## 📋 Reference Documents

Before starting implementation, review these planning documents:

1. **[PRD](file:///.gemini/antigravity/brain/4f5f27a6-635a-416f-8e82-d5fe0272b2bc/prd.md)** - Complete requirements and game rules
2. **[Roadmap](file:///.gemini/antigravity/brain/4f5f27a6-635a-416f-8e82-d5fe0272b2bc/roadmap.md)** - 8-week development plan

---

## 🚀 Phase 1: MVP Development (Weeks 1-3)

### Step 1: Project Initialization

**Prompt for Antigravity:**
```
Create a new Kachuful scoring app using Vite + React + TypeScript.

Requirements:
- Initialize project with: npx create-vite@latest ./ --template react-ts
- Setup Tailwind CSS for styling
- Create folder structure:
  /src
    /components  (UI components)
    /lib         (game logic)
    /types       (TypeScript types)
    /hooks       (custom React hooks)
    /styles      (global styles)
- Create design system with tokens for colors, spacing, typography

Follow web application best practices with premium, modern aesthetics.
```

**Expected Deliverables:**
- Working React + TypeScript + Tailwind setup
- Folder structure created
- Initial `design-tokens.css` or equivalent

---

### Step 2: Core Game Logic

**Prompt for Antigravity:**
```
Implement the core Kachuful game logic in /src/lib/game-logic.ts

Requirements:
1. Create TypeScript types/interfaces:
   - Player
   - Round
   - GameState
   - ScoringVariant

2. Implement functions:
   - calculateScore(predicted: number, actual: number, variant: ScoringVariant): number
   - validateBids(bids: number[], totalTricks: number, dealerIndex: number): boolean
   - getTrumpSuit(roundNumber: number): 'spades' | 'diamonds' | 'clubs' | 'hearts' | 'none'
   - getCardsDealt(roundNumber: number, totalRounds: number): number
   - isGameComplete(currentRound: number, totalRounds: number): boolean

3. Default scoring: 10 + predicted if exact match, 0 if wrong
4. Dealer restriction: Total bids cannot equal total tricks available
5. Trump rotation: ♠️ → ♦️ → ♣️ → ♥️ → No Trump (cycles)

Write comprehensive unit tests for all logic.
```

**Expected Deliverables:**
- `game-logic.ts` with all functions
- Type definitions in `types/game.ts`
- Unit tests passing

---

### Step 3: Game Setup Screen

**Prompt for Antigravity:**
```
Create the Game Setup screen component in /src/components/GameSetup.tsx

Requirements:
1. Form to collect:
   - Number of players (2-10, dropdown or stepper)
   - Player names (dynamic input fields)
   - Scoring variant (default: "10 + Predicted")
   - Optional: Starting hand size, total rounds

2. Validation:
   - All players must have names
   - No duplicate names
   - At least 2 players

3. Design:
   - Card-game aesthetic with card suit icons
   - Premium colors: deep blue/purple gradients with gold accents
   - Glassmorphism effects
   - Large, touch-friendly inputs for mobile
   - Smooth micro-animations on interactions

4. On submit: Initialize game state and navigate to bidding screen

Use Tailwind CSS and follow modern web design principles.
```

**Expected Deliverables:**
- `GameSetup.tsx` component
- Form validation working
- Beautiful, responsive UI

---

### Step 4: Bidding Screen

**Prompt for Antigravity:**
```
Create the Bidding Screen in /src/components/BiddingScreen.tsx

Requirements:
1. Display current round info:
   - Round number (e.g., "Round 3 of 13")
   - Cards dealt this round
   - Trump suit (with colored suit icon)
   - Current dealer

2. Bid entry for each player:
   - Show player name
   - Buttons 0-[max tricks] for bid selection
   - Highlight selected bid
   - Disable invalid bids for dealer (dealer restriction)

3. Validation:
   - Show warning if dealer's bid makes total = total tricks
   - All players must bid before proceeding

4. Premium design:
   - Large bid buttons (0-13)
   - Smooth animations when selecting
   - Progress indicator (3 of 4 players bid)
   - Visual feedback for dealer restriction

5. On complete: Move to tricks entry screen

Make it mobile-first with excellent touch targets.
```

**Expected Deliverables:**
- `BiddingScreen.tsx` component
- Dealer restriction validation working
- Polished UI with animations

---

### Step 5: Tricks Entry & Scoring

**Prompt for Antigravity:**
```
Create the Tricks Entry screen in /src/components/TricksEntry.tsx

Requirements:
1. After round is played physically, players enter tricks won
2. For each player:
   - Show name, bid amount
   - Stepper or number input for tricks won (0-[cards dealt])
   - Visual indicator if prediction matched

3. Auto-calculate scores when all tricks entered:
   - Use scoring algorithm from game-logic.ts
   - Show individual round scores
   - Update cumulative scores

4. Display:
   - Success indicator (green checkmark) if bid matched
   - Failure indicator (red X) if bid missed
   - Points earned this round

5. Action buttons:
   - "Next Round" → move to next bidding round
   - "View Scoreboard" → see full scores

Premium design with celebratory animations for successful predictions.
```

**Expected Deliverables:**
- `TricksEntry.tsx` component
- Score calculation integrated
- Smooth transitions

---

### Step 6: Scoreboard

**Prompt for Antigravity:**
```
Create the Scoreboard component in /src/components/Scoreboard.tsx

Requirements:
1. Table layout showing:
   - Player names (column headers or row headers)
   - Each round as row/column
   - Bid vs Actual for each round (e.g., "3/3 ✓" or "2/5 ✗")
   - Round score for each player
   - Cumulative total score (bold/highlighted)

2. Visual features:
   - Highlight current leader (gold border/background)
   - Color-code successful vs failed predictions
   - Responsive table (horizontal scroll on mobile if needed)
   - Sort by rank option

3. Summary stats:
   - Total rounds played
   - Rounds remaining
   - Current leader

4. Actions:
   - Continue to next round
   - View detailed breakdown
   - End game (if final round)

Use premium table design with card-game theming.
```

**Expected Deliverables:**
- `Scoreboard.tsx` component
- Responsive table layout
- Leader highlighting

---

### Step 7: Game State Management

**Prompt for Antigravity:**
```
Implement game state management using React Context or Zustand

Requirements:
1. Create global game state including:
   - Players array
   - Current round number
   - Rounds array (history of bids/tricks/scores)
   - Game settings (scoring variant, total rounds)
   - Game status (setup, bidding, playing, complete)

2. Actions:
   - initializeGame(settings)
   - submitBids(bids)
   - submitTricks(tricks)
   - nextRound()
   - endGame()
   - resetGame()

3. Persistence:
   - Save state to localStorage on every update
   - Load state on app mount
   - Handle corrupted data gracefully

4. Type safety:
   - Full TypeScript types for state and actions
   - Immutable updates

Choose between Context API or Zustand based on complexity.
```

**Expected Deliverables:**
- State management setup
- LocalStorage persistence working
- Type-safe state updates

---

### Step 8: Routing & Integration

**Prompt for Antigravity:**
```
Integrate all components with routing using React Router

Requirements:
1. Routes:
   / → Home/Game Setup
   /bidding → Bidding Screen
   /tricks → Tricks Entry
   /scoreboard → Scoreboard
   /results → Final Results (winner announcement)

2. Navigation guards:
   - Redirect to setup if no active game
   - Prevent accessing future rounds

3. Layout:
   - Shared header with game info (round, scores)
   - Navigation breadcrumbs
   - Exit/Quit game button

4. Transitions:
   - Smooth page transitions
   - Loading states where needed

Setup proper routing with state-aware navigation.
```

**Expected Deliverables:**
- React Router setup
- All screens connected
- Proper navigation flow

---

### Step 9: Winner Declaration

**Prompt for Antigravity:**
```
Create the Final Results screen in /src/components/FinalResults.tsx

Requirements:
1. Display after last round:
   - 🏆 Winner announcement with celebration
   - Final rankings (1st, 2nd, 3rd...)
   - Total scores for all players
   - Game statistics:
     - Most successful predictions
     - Highest single-round score
     - Average bid accuracy

2. Visual design:
   - Confetti or celebration animation
   - Trophy icon for winner
   - Medal icons for top 3
   - Premium card-reveal animation

3. Actions:
   - Save Game (to history)
   - Play Again (new game with same players)
   - New Game (fresh setup)
   - View Full Scoreboard

Make it feel rewarding with animations and effects.
```

**Expected Deliverables:**
- `FinalResults.tsx` component
- Winner celebration
- Game completion flow

---

### Step 10: MVP Testing & Polish

**Prompt for Antigravity:**
```
Polish the MVP for initial release:

1. Error Handling:
   - Add error boundaries
   - Handle invalid states gracefully
   - Show user-friendly error messages

2. Loading States:
   - Skeleton loaders where appropriate
   - Smooth transitions between screens

3. Responsive Design:
   - Test on mobile (375px)
   - Test on tablet (768px)
   - Test on desktop (1024px+)

4. Accessibility:
   - Add ARIA labels
   - Keyboard navigation support
   - Color contrast compliance

5. Performance:
   - Optimize animations (60 FPS)
   - Code splitting for routes
   - Image optimization

6. Deploy:
   - Build production bundle
   - Deploy to Vercel/Netlify
   - Test deployment

Ensure everything works smoothly end-to-end.
```

**Expected Deliverables:**
- Polished, production-ready MVP
- Deployed app with working URL
- Bug-free gameplay

---

## 🎨 Design Guidelines for Antigravity

When prompting Antigravity for UI components, always include these design requirements:

### Visual Aesthetics
- **Premium feel:** Vibrant gradients, glassmorphism, subtle shadows
- **Card game theme:** Use ♠️ ♥️ ♦️ ♣️ suit symbols as visual elements
- **Modern:** 2025-style design, not dated or basic
- **Dark mode:** Default to dark theme with rich colors

### Color Palette
- **Primary:** Deep blue (#1E3A8A) to purple (#7C3AED) gradient
- **Accent:** Gold (#F59E0B) for highlights and leader
- **Success:** Green (#10B981) for correct predictions
- **Error:** Red (#EF4444) for wrong predictions
- **Suits:** Red for ♥️♦️, Black for ♠️♣️

### Typography
- **Headings:** Bold, large (2xl-4xl)
- **Body:** Clean, readable (base-lg)
- **Numbers/Scores:** Monospace or tabular figures

### Animations
- **Micro-interactions:** Button hovers, clicks
- **Page transitions:** Smooth fade/slide
- **Celebration:** Confetti on winner reveal

---

## 📦 Phase 2-4: Future Enhancements

When ready to extend beyond MVP, use these prompts:

### Game History
```
Add game history feature:
- Save completed games to IndexedDB
- Show list of past games with date, players, winner
- View detailed scoreboard of past games
- Delete games
- Search/filter by player name or date
```

### Player Statistics
```
Create player statistics dashboard:
- Win rate per player
- Average score
- Bid accuracy (% of rounds where prediction matched)
- Best performance (highest score)
- Head-to-head records
- Charts using Chart.js or Recharts
```

### Dark Mode Toggle
```
Implement dark/light mode toggle:
- Use Tailwind dark: classes
- Persist preference in localStorage
- Smooth transition between modes
- Update all components for both themes
```

### Export & Sharing
```
Add export features:
- Export scoreboard as PDF (use jsPDF)
- Export as CSV for Excel
- Generate shareable link with game state
- Copy scoreboard to clipboard
```

### PWA Features
```
Convert to Progressive Web App:
- Add manifest.json
- Create service worker for offline support
- Enable "Add to Home Screen"
- Cache assets for performance
- Test offline functionality
```

---

## 🧪 Testing Prompts

### Unit Testing
```
Write comprehensive unit tests for game logic:
- Test all scoring variants
- Test dealer restriction validation
- Test trump rotation
- Test edge cases (0 bids, max bids)
- Use Vitest or Jest
- Aim for 90%+ code coverage
```

### Integration Testing
```
Add integration tests for user flows:
- Complete game flow (setup → bidding → tricks → results)
- State persistence and restoration
- Navigation between screens
- Error scenarios
- Use React Testing Library
```

---

## 🛠️ Troubleshooting

### Common Issues & Prompts

**Issue:** Scoring calculation incorrect
```
Debug the scoring calculation in game-logic.ts:
- Add detailed console logs for predicted vs actual
- Verify formula: 10 + predicted (if match), else 0
- Test with edge case: 0 predicted, 0 actual → should be 10 points
- Check for off-by-one errors
```

**Issue:** Mobile UI not responsive
```
Fix mobile responsiveness:
- Use Tailwind responsive classes (sm:, md:, lg:)
- Test bid buttons are min 44px touch targets
- Ensure tables scroll horizontally on small screens
- Check text sizes are readable (min 16px)
- Test on iPhone SE (375px width)
```

**Issue:** State not persisting
```
Debug localStorage persistence:
- Console log state before saving
- Check localStorage in DevTools
- Verify JSON serialization doesn't lose data
- Add error handling for QuotaExceeded
- Test state restoration on page reload
```

---

## 📱 Deployment Checklist

Before launching, verify with Antigravity:

```
Prepare for production deployment:

1. Build optimization:
   - Run 'npm run build'
   - Verify bundle size < 500KB
   - Check for console errors in production build

2. Performance:
   - Run Lighthouse audit (target 90+ score)
   - Optimize images (use WebP)
   - Enable code splitting

3. SEO & Meta:
   - Add proper title and meta description
   - Include Open Graph tags
   - Add favicon and app icons

4. Security:
   - No API keys exposed
   - Proper CORS if applicable
   - HTTPS enabled

5. Deploy to Vercel:
   - Connect GitHub repo
   - Configure deployment settings
   - Test production URL

6. Post-deployment:
   - Test on real devices
   - Share with beta users
   - Monitor for errors (add Sentry if needed)
```

---

## 🎯 Quick Command Reference

```bash
# Initialize project
npx create-vite@latest ./ --template react-ts

# Install dependencies
npm install

# Add Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Install additional packages
npm install react-router-dom zustand  # or use Context API
npm install lucide-react  # for icons

# Development
npm run dev

# Build
npm run build

# Preview build
npm run preview

# Deploy to Vercel
npx vercel

# Run tests
npm run test
```

---

## 📚 Additional Resources

Share these with Antigravity when relevant:

- [Kachuful Wikipedia](https://en.wikipedia.org/wiki/Kachuful)
- [React Documentation](https://react.dev)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)
- [Vite Guide](https://vitejs.dev/guide/)

---

## ✅ Success Criteria

Before considering MVP complete:

- [ ] Full game playable without errors
- [ ] Scoring 100% accurate (verified manually)
- [ ] Mobile responsive (tested on phone)
- [ ] Dark mode with premium aesthetics
- [ ] LocalStorage persistence working
- [ ] 5+ test games completed successfully
- [ ] Deployed and accessible via URL

---

## 🚀 Next Steps

1. Start with **Step 1: Project Initialization**
2. Work through steps sequentially
3. Test thoroughly after each step
4. Use this guide as reference throughout development
5. After MVP, proceed to Phase 2 enhancements

**Ready to begin? Start with the project initialization prompt above!**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dhawalhost)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dhawalhost)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
