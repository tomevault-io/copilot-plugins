## poker-hand-replay55665612sdfg

> Build a web application that allows live poker players to manually input their hand history and generate shareable video replays. The target audience is recreational and serious live poker players who want to review, share, and discuss hands from their sessions without the auto-export features available in online poker.

# Poker Hand Replay - Project Prompt

## Overview

Build a web application that allows live poker players to manually input their hand history and generate shareable video replays. The target audience is recreational and serious live poker players who want to review, share, and discuss hands from their sessions without the auto-export features available in online poker.

## Problem Statement

Live poker players have no easy way to visualize and share their hands. They currently resort to:
- Typing out hand histories in text (hard to follow)
- Drawing on napkins or whiteboards
- Using clunky desktop software not designed for mobile

We're building a mobile-first, fast input tool that outputs professional-looking video replays optimized for sharing on social media, Discord, and poker forums.

## Core User Flow

1. **Setup** → Configure the table (blinds, player count, positions, stack sizes)
2. **Hero Cards** → Input your hole cards
3. **Action Entry** → Log the hand action-by-action (who did what, for how much)
4. **Community Cards** → Input flop/turn/river as they come
5. **Replay** → Watch the animated replay
6. **Export** → Download as video (MP4/WebM) or share directly

## Technical Requirements

### Stack
- **Frontend**: React (single-page app)
- **Styling**: Tailwind CSS or CSS-in-JS (keep it minimal)
- **Video Export**: Client-side preferred (`html2canvas` + `ffmpeg.wasm` or `MediaRecorder` API)
- **No backend required** for MVP — all processing happens in browser
- **Mobile-first**: Must work well on phones since players input hands at the table

### Key Components

```
/components
  /Table           — Poker table visualization (SVG or Canvas)
  /Card            — Playing card component
  /PlayerSeat      — Player position with stack, cards, action badge
  /ActionTimeline  — Visual timeline of the hand
  /InputForm       — Manual entry forms for setup/actions
  /ReplayControls  — Play/pause/step/speed controls
  /ExportModal     — Video export options and progress
```

### Data Model

```typescript
interface HandHistory {
  id: string;
  blinds: { sb: number; bb: number; ante?: number };
  players: Player[];
  heroPosition: Position;
  communityCards: Card[];
  actions: Action[];
  createdAt: Date;
}

interface Player {
  position: Position;
  stack: number;
  cards: [Card | null, Card | null];
  name?: string;
  isHero: boolean;
}

interface Action {
  street: 'preflop' | 'flop' | 'turn' | 'river';
  player: Position;
  type: 'fold' | 'check' | 'call' | 'bet' | 'raise' | 'all-in';
  amount?: number;
  isAllIn?: boolean;
}

interface Card {
  rank: 'A' | 'K' | 'Q' | 'J' | 'T' | '9' | '8' | '7' | '6' | '5' | '4' | '3' | '2';
  suit: '♠' | '♥' | '♦' | '♣';
}

type Position = 'BTN' | 'SB' | 'BB' | 'UTG' | 'UTG+1' | 'MP' | 'MP+1' | 'HJ' | 'CO';
```

## Design Direction

### Aesthetic: Minimalist Dark

- **Theme**: Dark background (#0a0a0a), subtle borders, no gradients except table felt
- **Typography**: Monospace for numbers/cards (JetBrains Mono), clean sans-serif for UI (Inter)
- **Colors**:
  - Background: `#0a0a0a`
  - Surface: `#111111`, `#1a1a1a`
  - Border: `#222222`, `#333333`
  - Primary accent: `#2563eb` (blue)
  - Card red: `#dc2626`
  - Card black: `#171717`
  - Bet/raise: `#22c55e` (green)
  - Fold: `#ef4444` (red)
  - All-in: `#f59e0b` (amber)
  - Pot: `#fbbf24` (gold)

### Table Design
- Oval felt with subtle gradient (dark green)
- Simple wood-tone rail
- Clean player boxes with position labels
- No skeuomorphic chips — just numbers
- Cards: Clean, flat design with clear rank/suit

### Animation Philosophy
- Smooth but fast — players want to get to the export quickly
- Key animations:
  - Card dealing (slide in)
  - Community cards (flip reveal)
  - Action badges (fade in)
  - Pot updates (number tick up)
- No excessive flourishes

## Input UX Priorities

### Speed is Everything
- Minimize taps/clicks to log an action
- Smart defaults (auto-advance to next player, remember common bet sizes)
- Support swipe gestures for common actions on mobile

### Card Input
- Visual card picker (tap suit, then rank)
- Or keyboard shortcut: type "Ah" for A♥, "Ks" for K♠
- Recent cards shown for quick re-selection

### Action Input
- Big touch targets for fold/check/call/bet/raise
- Number pad for bet amounts
- Quick buttons for common sizes (1/3 pot, 1/2 pot, pot, 2x)

## Video Export Requirements

### Output Specs
- **Format**: MP4 (H.264) primary, WebM fallback
- **Resolution**: 1080x1080 (square, optimal for Instagram/Twitter) or 1920x1080 (landscape)
- **Duration**: ~1-2 seconds per action, user-adjustable speed
- **File size**: Target under 10MB for easy sharing

### Export Options
- Playback speed (0.5x, 1x, 1.5x, 2x)
- Include/exclude hero cards reveal at end
- Add title card with hand summary
- Watermark toggle (for branding if productized)

### Technical Approach (Client-side)
1. Render each frame to canvas using the table component
2. Use `MediaRecorder` API to capture canvas as video stream
3. Or use `ffmpeg.wasm` to encode frames into MP4
4. Provide download link when complete

## Future Features (Post-MVP)

- **Hand sharing**: Generate shareable links (requires backend)
- **Hand library**: Save and organize hands locally (IndexedDB)
- **Equity calculator**: Show winning percentages at each street
- **Range visualization**: Display opponent ranges based on actions
- **Voice input**: "Button raises to 15" → auto-logs action
- **Multi-hand sessions**: Log entire sessions, tag interesting hands
- **Social features**: Comments, likes, follows (requires backend)
- **Embed widget**: Embed replays on forums/blogs

## File Structure

```
poker-hand-replay/
├── src/
│   ├── components/
│   │   ├── Table/
│   │   │   ├── Table.tsx
│   │   │   ├── Felt.tsx
│   │   │   ├── PlayerSeat.tsx
│   │   │   └── CommunityCards.tsx
│   │   ├── Card/
│   │   │   ├── Card.tsx
│   │   │   └── CardPicker.tsx
│   │   ├── Input/
│   │   │   ├── SetupForm.tsx
│   │   │   ├── ActionInput.tsx
│   │   │   └── QuickBetButtons.tsx
│   │   ├── Replay/
│   │   │   ├── ReplayView.tsx
│   │   │   ├── ReplayControls.tsx
│   │   │   └── ActionTimeline.tsx
│   │   └── Export/
│   │       ├── ExportModal.tsx
│   │       └── VideoRenderer.tsx
│   ├── hooks/
│   │   ├── useHandHistory.ts
│   │   ├── useReplay.ts
│   │   └── useVideoExport.ts
│   ├── utils/
│   │   ├── poker.ts          # Pot calculations, hand evaluation
│   │   ├── positions.ts      # Position logic for N players
│   │   └── export.ts         # Video encoding utilities
│   ├── types/
│   │   └── index.ts
│   ├── App.tsx
│   └── main.tsx
├── public/
├── package.json
├── tailwind.config.js
├── tsconfig.json
└── README.md
```

## Development Priorities

### Phase 1: Core Input & Visualization
- [ ] Table component with dynamic player positions
- [ ] Card picker component
- [ ] Setup form (blinds, players, stacks)
- [ ] Action logging form
- [ ] Basic replay with play/pause

### Phase 2: Polish & Animation
- [ ] Smooth card/action animations
- [ ] Pot calculation display
- [ ] Action badges with amounts
- [ ] Street transitions (flop/turn/river reveals)

### Phase 3: Video Export
- [ ] Canvas-based rendering
- [ ] MediaRecorder integration
- [ ] Export progress UI
- [ ] Download functionality

### Phase 4: Mobile Optimization
- [ ] Touch-optimized inputs
- [ ] Swipe gestures
- [ ] Responsive table scaling
- [ ] PWA support for offline use

## Key Principles

1. **Fast input > pretty input**: Players are at the table, possibly between hands. Every tap counts.

2. **Replay clarity > realism**: The goal is to communicate what happened, not simulate a real table. Keep it clean and readable.

3. **Shareable by default**: Every design decision should consider how it looks when shared on social media.

4. **Offline-first**: Works without internet. All processing client-side.

5. **No account required**: Users can start using immediately. Accounts only needed for cloud features later.

## Example Hand for Testing

```
Blinds: $1/$2
Players: 6-handed
Hero: BTN with A♠K♥

Preflop:
- UTG folds
- MP raises to $6
- HJ folds
- CO folds
- BTN (Hero) raises to $18
- SB folds
- BB folds
- MP calls $12

Flop: K♠ 7♦ 2♣
- MP checks
- BTN bets $22
- MP calls $22

Turn: 5♥
- MP checks
- BTN bets $55
- MP folds

Hero wins $85 pot
```

Use this hand to test the full flow from input to video export.

## Poker Basics (Texas Hold'em Rules)

This app is built around **No-Limit Texas Hold'em (NLHE)**, the most common live poker format.

### Objective

Win the pot by either:

1. making the best 5-card poker hand at showdown, or
2. getting all opponents to fold before showdown.

### Cards & Hand Construction

* Each player receives **2 private hole cards**.
* Up to **5 community cards** are dealt face up on the board.
* Your final hand is the **best 5-card combination** made from **any** of your 2 hole cards + the 5 board cards.

### Hand Rankings (High → Low)

1. Royal Flush (A-K-Q-J-T same suit)
2. Straight Flush
3. Four of a Kind
4. Full House
5. Flush
6. Straight
7. Three of a Kind
8. Two Pair
9. One Pair
10. High Card

### Betting Streets (Rounds)

A hand is played across up to four streets:

1. **Preflop**: hole cards are dealt, then betting starts
2. **Flop**: 3 community cards dealt, then betting
3. **Turn**: 1 community card dealt, then betting
4. **River**: 1 community card dealt, then final betting
   If multiple players remain after the river betting round, the hand goes to showdown.

### Blinds & Antes

* Before cards are dealt:

  * **Small Blind (SB)** posts the small blind amount
  * **Big Blind (BB)** posts the big blind amount
  * Optional: **Ante** may be posted by all players (or BB ante format, depending on game)
* These forced bets create the initial pot.

### Action Types (What a player can do)

Depending on the situation:

* **Fold**: give up the hand (cannot win the pot)
* **Check**: bet 0 (only allowed if no bet has been made in the current round)
* **Call**: match the current bet
* **Bet**: put chips in when no bet exists yet in this round
* **Raise**: increase the current bet amount
* **All-in**: wager all remaining chips (treated as a bet/raise/call depending on context)

### Turn Order

* **Preflop**: action starts from the player **left of the BB** (UTG in 6-max/9-max).
* **Postflop (flop/turn/river)**: action starts from the **SB** (or the leftmost remaining player) and proceeds clockwise.
* The **dealer button (BTN)** rotates each hand.

### Betting Rules (No-Limit)

* Players may bet/raise any amount up to their full stack.
* A betting round ends when:

  * all remaining players have either folded, or
  * all remaining players have contributed the same amount for that round (i.e., everyone has called/checked).

### Showdown

* If two or more players remain after the final betting round:

  * hands are revealed (or mucked if allowed)
  * the best 5-card hand wins the pot
* Ties split the pot.

### All-in & Side Pots

* If a player goes **all-in** for less than the full bet:

  * they can only win the portion of the pot they are eligible for
  * additional chips from other players form one or more **side pots**
* Side pots are awarded among players who contributed to them and did not fold.

### Notes for This App (Replay/Validation)

* The board must contain **5 unique cards** maximum (3 flop + 1 turn + 1 river).
* No card can appear more than once anywhere (hero hole cards cannot be re-selected on the board).
* Street order is fixed: **preflop → flop → turn → river**.
* Actions should be validated by street: e.g., community cards cannot be placed before the flop, and "check" is only allowed if no bet exists in the current betting state.
* The order of action must be clockwise

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zackhsu5566)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Zackhsu5566)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
