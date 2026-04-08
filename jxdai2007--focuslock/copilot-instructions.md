## focuslock

> FocusLock is an AI-powered study accountability web app for a 12-hour hackathon (HOTH 12, UCLA). It uses the webcam + Gemini Vision API to detect when a student loses focus, penalizes them with a lives system, and gamifies studying with streaks, coins, and AI roasts.

# CLAUDE.md — FocusLock

## What This Is
FocusLock is an AI-powered study accountability web app for a 12-hour hackathon (HOTH 12, UCLA). It uses the webcam + Gemini Vision API to detect when a student loses focus, penalizes them with a lives system, and gamifies studying with streaks, coins, and AI roasts.

## Tech Stack
- **Framework**: Next.js 14 (App Router), TypeScript
- **Styling**: Tailwind CSS + shadcn/ui (dark mode, zinc theme)
- **Animations**: Framer Motion (transitions, toasts, number animations) + lottie-react (flame, coin)
- **Charts**: Recharts (focus timeline)
- **Sound**: Howler.js
- **State**: Zustand with localStorage persist
- **AI**: Gemini 2.0 Flash vision API (free tier, 15 RPM)
- **Deploy**: Vercel

## Design System — "Dark Ember"
- Background: #0a0a0a with subtle radial ember glow
- Glass cards: `backdrop-blur-md bg-zinc-900/40 border border-zinc-800/50 rounded-2xl`
- Flame color states: amber (#f59e0b) idle → orange-red (#ef4444) easy → purple-blue (#7c3aed → #3b82f6) hard
- Distracted: flame shrinks/dims. Life lost: red flash + shake. Focused streak: flame grows.
- Display/game font: Orbitron (Google Font) for scores, streaks, coins, headings
- Body font: Geist Sans (default Next.js)
- NO generic AI aesthetics. Dark, gamified, ember/fire theme throughout.

## Core UX Flow
1. **Open app** → dark screen, flame ignites center, idle state with streak/history
2. **Start session** → setup modal: task description, session length, difficulty (lives), site blocking toggle
3. **Session active** → flame = focus gauge, webcam captures every 12s, Gemini analyzes, lives system
4. **Distraction detected** → lose a life, burning sound, flame sputters, AI roast toast
5. **Session end** → summary with grade (S/A/B/C), stats, AI review, coins earned
6. **Between sessions** → view history, stats, day streak, spend coins in shop

## Key Architecture Decisions
- Webcam captures a frame every 12 seconds (6 RPM, well under Gemini's 15 RPM limit)
- Gemini prompt includes session context (task description, lives remaining, streak, distraction count) so roasts are adaptive and escalating
- Task description tells AI what devices are acceptable (e.g., iPad for art homework = not a distraction)
- Tab visibility API detects when user leaves the tab → counts as distraction
- Only counts a NEW distraction when previous state was "focused" (prevents double-penalizing)
- Coins formula: base 10 + 1/min focused + 5/life remaining + 10 if >80% focus + 20 if no lives lost

## Gemini API Details
- Model: gemini-2.0-flash
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- API key: NEXT_PUBLIC_GEMINI_API_KEY from .env.local
- Response format: JSON with `responseMimeType: "application/json"`
- Expected response shape:
```json
{
  "status": "focused" | "distracted" | "away",
  "distraction_type": "phone" | "sleeping" | "chatting" | "zoned_out" | "eating" | "looking_away" | null,
  "confidence": 0.0-1.0,
  "roast": "1-2 sentence message"
}
```

## Project Structure
```
app/              → layout.tsx, page.tsx, globals.css
components/
  flame/          → FocusFlame.tsx (central Lottie flame, Fire.json is here)
  gamification/   → CoinCounter.tsx, MilestoneToast.tsx, Shop.tsx (coin.json is here)
  history/        → SessionHistory.tsx
  roast/          → RoastToast.tsx
  session/        → SessionSetup.tsx, SessionActive.tsx, SessionSummary.tsx, LivesDisplay.tsx
  stats/          → StatsPanel.tsx, FocusTimeline.tsx, StreakBadge.tsx
  webcam/         → WebcamCapture.tsx
lib/              → gemini.ts, focusEngine.ts, types.ts, sounds.ts
stores/           → sessionStore.ts (Zustand)
public/
  animations/     → (Lottie JSONs if moved here)
  sounds/         → burn.mp3, coin.mp3, ping.mp3, fanfare.mp3
```

## Rules for Claude Code
- Always use TypeScript with strict types. Define interfaces in lib/types.ts.
- All styling with Tailwind + shadcn. Use the glass-card pattern for containers.
- Use Orbitron font for ALL numbers, scores, counters, and game-like headings.
- Use Framer Motion for all animations (entrances, exits, number transitions, shakes).
- Sound effects via Howler.js — always wrap in try/catch, never break the app if sound fails.
- Keep Gemini API calls in lib/gemini.ts only. Everything else gets data through the Zustand store.
- Focus engine functions in lib/focusEngine.ts must be PURE (no side effects).
- When building UI components, make them feel like a game dashboard, not a generic SaaS app.
- Dark theme always. No white backgrounds. No light mode.   

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jxdai2007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jxdai2007)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
