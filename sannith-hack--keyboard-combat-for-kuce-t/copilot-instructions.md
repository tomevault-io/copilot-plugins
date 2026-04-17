## keyboard-combat-for-kuce-t

> **Keyboard Combat** is a scalable typing competition web application designed for high-stakes events (e.g., college fests). It features a gamified multi-level typing experience, real-time leaderboards, and a comprehensive admin dashboard for competition management.

# Keyboard Combat - Project Context

## 🎯 Project Overview
**Keyboard Combat** is a scalable typing competition web application designed for high-stakes events (e.g., college fests). It features a gamified multi-level typing experience, real-time leaderboards, and a comprehensive admin dashboard for competition management.

### 🚀 Tech Stack
- **Frontend:** React (Vite) + TypeScript
- **Styling:** Tailwind CSS (v4) + PostCSS
- **State Management:** Zustand (with persistence)
- **Routing:** React Router DOM
- **Backend/DB:** Supabase (Database + Realtime)
- **Icons:** Lucide React

---

## 🏗️ Architecture & Core Logic

### Competition Flow
The application follows a structured flow managed in `src/App.tsx`:
0. **Entry:** Participant registration (allows early entry for upcoming events).
1. **Waiting Room:** Countdown timer holding area until the competition goes live.
2. **Level 1 (Paragraphs):** Natural language typing.
3. **Break:** 10-second intermission.
4. **Level 2 (Code):** C/C++ snippet typing (strict character comparison).
5. **Break:** 10-second intermission.
6. **Level 3 (Precision):** High-precision special characters and symbols.
7. **Results:** Final performance breakdown and database submission.

### 🧠 Scoring Schema (Combat Points)
The application uses a weighted scoring formula to prioritize accuracy over raw speed:
`Combat Score = rawWpm * (Accuracy / 100)^2`
- **WPM:** Words Per Minute calculated as `(TypedChars / 5) / Minutes`.
- **Accuracy:** Percentage of correct characters.
- **Precision:** All stats are tracked to 2 decimal places.

### 🛡️ Anti-Cheat Mechanisms
- **Tab Switching:** Detects `visibilitychange` and alerts the user (potential for auto-disqualification).
- **Clipboard Lock:** Disables copy, paste, and right-click context menus.
- **Focus Guard:** Text input is hidden; users type into a transparent textarea over the rendered text.

---

## 📊 Database Schema (Supabase)
The project uses four primary tables:
1. **`competitions`**: Manages event slots (id, name, status [draft/live/ended], scheduled_start).
2. **`participants`**: Stores student details (id, name, roll_number, email, college, competition_id).
3. **`attempts`**: Level-by-level performance (id, participant_id, level, wpm [NUMERIC], accuracy [NUMERIC], time_taken, combat_score [NUMERIC], competition_id).
4. **`results`**: Final aggregated scores for the leaderboard (id, participant_id, level1_wpm, level2_wpm, level3_wpm, avg_wpm, avg_accuracy, total_score, competition_id).

---

## ⚙️ Development Workflows

### Environment Variables
Required variables in `.env`:
- `VITE_SUPABASE_URL`: Supabase project URL.
- `VITE_SUPABASE_ANON_KEY`: Supabase anonymous API key.
- `VITE_ADMIN_USER`: Username for admin dashboard.
- `VITE_ADMIN_PASS`: Password for admin dashboard.

### Commands
- `npm run dev`: Starts the Vite development server.
- `npm run build`: Compiles TypeScript and builds the production bundle.
- `npm run preview`: Locally previews the production build.

### Admin Dashboard
Access via `/login` or `/admin`.
- **Schedule:** Create and manage competition slots.
- **Auto-Live:** Scheduled competitions automatically transition to 'live' when their `scheduled_start` time is reached.
- **Live Control:** "GO LIVE" and "END" buttons for manual override.
- **Monitoring:** Real-time leaderboard updates via Supabase Realtime subscriptions, tracking precise Combat Points and Average Accuracy.

---

## 📝 Conventions
- **State:** Use `src/store/useGameStore.ts` for all shared application state.
- **Components:** Modularize UI in `src/components/`.
- **Data:** Typing texts are stored in `src/data/content.ts`.
- **Types:** Strictly type all Supabase responses and store actions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sannith-Hack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
