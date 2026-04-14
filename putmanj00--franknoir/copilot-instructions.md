## franknoir

> You are working on **FrankNoir**, a Valentine's Day scavenger hunt experience that recreates 14 years of relationship history across Cincinnati locations.

# FrankNoir — Valentine's Day Scavenger Hunt

You are working on **FrankNoir**, a Valentine's Day scavenger hunt experience that recreates 14 years of relationship history across Cincinnati locations.

## Quick Reference

- **PRD/Stories**: `prd.json`
- **Progress Log**: `progress.txt`
- **Plan Documents**: `planDocuments/`
- **Vision Docs**: `planDocuments/*.md`
- **Timeline**: 6 days until Valentine's Day (Feb 14, 2026)

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Node.js |
| Framework | Next.js 16 (App Router) |
| Language | TypeScript (strict mode) |
| UI Library | HeroUI |
| Styling | Tailwind CSS |
| State | React Context + localStorage |
| Location | Geolocation API (browser native) |
| Audio | Web Audio API |
| Maps | Optional (Leaflet or Google Maps JS) |
| Hosting | Vercel (free tier) |

## Design System: Lisa Frank Noir

### Color Tokens
```css
--void-black: #030303;         /* Background */
--neon-magenta: #f53fe8;       /* Primary accent, borders */
--lisa-frank-purple: #ac3cfe;  /* Completed status */
--solar-flare-gold: #ffb622;   /* Active status, warnings */
--acid-yellow: #fffd42;        /* Focus rings, data input */
```

### Component Patterns
- **Cards**: Black background + 1px neon magenta stroke + glassmorphism blur
- **Typography**: Monospaced font (JetBrains Mono or Fira Code)
- **Buttons**: Purple→Magenta gradient for primary actions
- **Status**: HeroUI Chip components (Grey/Gold/Purple)
- **Layout**: Vertical scrolling timeline (mobile-first)

## Project Structure

```
frankNoir/
├── src/
│   ├── app/                    # Next.js 16 App Router
│   │   ├── page.tsx           # Timeline dashboard (main screen)
│   │   ├── stage/[id]/        # Individual stage pages
│   │   └── layout.tsx         # Root layout with Lisa Frank Noir theme
│   ├── components/
│   │   ├── StageCard.tsx      # Timeline card with status chips
│   │   ├── GPSTracker.tsx     # Geolocation hook + unlock logic
│   │   ├── RadioPuzzle.tsx    # Audio frequency slider (Stage 03)
│   │   ├── CipherInput.tsx    # Damascus cipher entry (Stage 08)
│   │   ├── ScanButton.tsx     # Placeholder "Scan" UI (MVP version)
│   │   └── Countdown.tsx      # Timer for time-sensitive stages
│   ├── lib/
│   │   ├── stages.ts          # Stage data: coords, puzzles, clues
│   │   ├── gps.ts             # Distance calculation, proximity checks
│   │   └── storage.ts         # localStorage helpers for progress
│   └── styles/
│       └── lisaFrankNoir.css  # Color tokens, glassmorphism
├── public/
│   ├── audio/                 # Stage audio files
│   ├── images/                # Stage visuals
│   └── manifest.json          # PWA manifest
├── planDocuments/             # Vision docs
├── prd.json                   # Story tracking
├── progress.txt               # Implementation log
└── CLAUDE.md                  # This file
```

---

## Story Workflow (Ralph Loop)

When working on stories, follow this workflow:

### 1. Understand the Work

```bash
# Check current story status
cat prd.json | jq '.epics[] | select(.stories[].passes == false)'

# Review recent progress
tail -100 progress.txt
```

### 2. Implement the Story

- Follow the Lisa Frank Noir design system
- Keep changes focused on the current story
- Reference `planDocuments/` for original requirements
- Mobile-first responsive design
- Test on actual phone when possible

### 3. Manual Testing

Given the 6-day timeline, focus on manual testing:

**For UI changes:**
1. Load the page in a browser
2. Test on mobile viewport (Chrome DevTools)
3. Verify Lisa Frank Noir styling is correct
4. Test user interactions

**For GPS features:**
1. Use mock mode (?mock=true) for development
2. Field test at 2-3 real locations before launch
3. Verify unlock radius is appropriate

**For puzzles:**
1. Solve the puzzle manually
2. Test edge cases (wrong answers)
3. Verify hint system works

### 4. Commit with Formatted Message

```
[E{epic}-S{story}] Brief description

- Key change 1
- Key change 2

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### 5. Update Tracking Files

**Update `prd.json`:**
- Set `passes: true` for the completed story

**Append to `progress.txt`:**
```markdown
## [Date] - Story #[ID]: [Title]

**Implementation**:
- What was done
- Key technical decisions

**Files Changed**:
- path/to/file.ts - what changed

**Testing**:
- How it was tested
- Any issues found

**Learnings**:
- Patterns for future iterations
- Gotcas to avoid
```

---

## Architecture Principles

1. **Client-only architecture** — No backend, all logic in browser
2. **localStorage for persistence** — Progress survives page refresh
3. **Sequential unlocking** — Must complete Stage N to unlock Stage N+1
4. **GPS-based progression** — Stages unlock when within radius
5. **Progressive hints** — 3 levels: subtle → clear → obvious
6. **Mock mode for testing** — ?mock=true bypasses GPS
7. **PWA installable** — Home screen icon, works offline

## Code Patterns

### Stage Data Structure
```typescript
interface Stage {
  id: number;
  title: string;
  location: string;
  coordinates: { lat: number; lng: number };
  time: string;
  description: string;
  clue: string;
  unlockType: 'gps' | 'puzzle' | 'scan';
  unlockRadius: number; // meters
  hints: [string, string, string]; // 3 levels
  completed: boolean;
}
```

### GPS Distance Calculation
```typescript
// Haversine formula for GPS distance
function getDistanceInMeters(lat1: number, lon1: number, lat2: number, lon2: number): number {
  const R = 6371e3; // Earth radius in meters
  const φ1 = lat1 * Math.PI / 180;
  const φ2 = lat2 * Math.PI / 180;
  const Δφ = (lat2 - lat1) * Math.PI / 180;
  const Δλ = (lon2 - lon1) * Math.PI / 180;
  const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
            Math.cos(φ1) * Math.cos(φ2) *
            Math.sin(Δλ/2) * Math.sin(Δλ/2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return R * c;
}
```

### localStorage Helpers
```typescript
// Save progress
localStorage.setItem('frankNoir:progress', JSON.stringify(stages));

// Load progress
const saved = localStorage.getItem('frankNoir:progress');
const stages = saved ? JSON.parse(saved) : DEFAULT_STAGES;
```

---

## GPS Testing Strategy

**Development:**
- Mock mode: `?mock=true` URL parameter simulates GPS movement
- Admin override: Manual unlock button (dev only)
- Generous radius: Start with 50m (adjust after field testing)

**Field Testing (Day 5):**
- Drive-by 3-4 key locations
- Test GPS trigger accuracy
- Adjust radiuses based on real behavior
- Test on her actual phone model

**Day-of Contingencies:**
- USB power bank (battery backup)
- Printable clue map (tech failure fallback)
- Manual unlock codes (GPS inaccuracy backup)
- Your phone as backup (text hints)

---

## Current Focus

**Target**: Valentine's Day 2026-02-14 (6 days away)
**Strategy**: MVP first, enhancements after

### Phase 1: MVP (Days 1-5)
- ✅ Timeline dashboard
- ✅ GPS-triggered unlocking
- ✅ Interactive puzzles
- ✅ Countdown timers
- ✅ Lisa Frank Noir UI
- ✅ 3-level hint system

### Phase 2: Post-Valentine's (Day 6+)
- ⏸️ Real AR.js integration
- ⏸️ Advanced animations
- ⏸️ Photo uploads
- ⏸️ Replay mode

Check `prd.json` for the next incomplete story (`passes: false`).

---

## Important Notes

- Always read the story from `prd.json` before starting
- Check `progress.txt` for context from previous work
- Test on mobile viewport (this is mobile-first)
- Update both `prd.json` AND `progress.txt` after completing a story
- Reference `planDocuments/` for vision and requirements
- GPS coordinates will be researched in Story E3-S1

---

## MVP vs. Full Vision

### Include in MVP (Valentine's Day)
- Timeline dashboard with 15 stages
- GPS-triggered unlocking
- Text-based clues and narrative
- Simple "Scan" button (not real AR)
- Web audio for radio puzzle
- 3-level hint system
- Countdown timers

### Defer to Post-Launch
- Real AR.js scanning
- Advanced camera features
- Complex animations
- Photo uploads
- Analytics dashboard

---

## Common Pitfalls & Learnings

### GPS
- **Accuracy varies**: Indoor GPS is unreliable (use generous radius)
- **Permission flow**: Must explain why GPS is needed upfront
- **Battery drain**: Test power consumption on long sessions
- **Fallback required**: Always have manual unlock codes

### PWA
- **iOS Safari quirks**: Test Add to Home Screen flow
- **Offline mode**: Cache critical assets
- **Storage limits**: localStorage has 5-10MB limit

### Timeline Pressure
- **Scope creep**: Stick to MVP, resist feature additions
- **Perfect is the enemy of good**: Functional > beautiful
- **Field testing critical**: Can't simulate GPS at desk

---

## Completion Signal

After updating the PRD:
- If ALL stories now have `passes: true`, reply ONLY with: `<promise>COMPLETE</promise>`
- Otherwise, continue with normal response (the script will run you again)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/putmanj00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/putmanj00)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
