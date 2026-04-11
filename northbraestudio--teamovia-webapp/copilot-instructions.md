## teamovia-webapp

> Create a data-agnostic "Evidence Dashboard" that can be populated with tactical anomalies from any match. The system will demonstrate TAS (Team Aptitude Score) and SynIQ using a scalable database structure.

# Teamovia AI - Multi-Team Prototype Build Plan

## 🎯 Objective
Create a data-agnostic "Evidence Dashboard" that can be populated with tactical anomalies from any match. The system will demonstrate TAS (Team Aptitude Score) and SynIQ using a scalable database structure.

## 🛠 Tech Stack
- **Frontend:** Next.js, Tailwind CSS, Lucide Icons.
- **Backend/DB:** Supabase (PostgreSQL).
- **Deployment:** Vercel.

## 🗄️ Scalable Database Schema (Supabase)
To ensure the app isn't "hardcoded," we will use the following relational structure:

1. **Teams Table:** `id, name, logo_url, primary_color`.
2. **Matches Table:** `id, home_team_id, away_team_id, date, video_url (YouTube/Vimeo)`.
3. **Tactical_Events Table:** - `id, match_id, timestamp_seconds, title` (e.g., "Defensive Gap").
   - `event_type` (Anomaly vs Benchmark).
   - `visual_data` (JSON field containing player coordinates for the SVG pitch).
   - `insight_text` (The "Explainable AI" coaching output).
   - `tas_score` (The 0-100 metric for that moment).

## 📅 14-Day Sprint (Agnostic Build)

### Phase 1: The Data Foundation (Days 1-3)
- [ ] Initialize Supabase and run SQL migrations for the generic tables above.
- [ ] Create a simple "Admin" form or use Supabase Dashboard to upload event data for *any* test match.
- [ ] **UK English Check:** Ensure all UI labels (e.g., "Authorised," "Licence," "Defence") use UK spelling.

### Phase 2: The Universal Pitch Engine (Days 4-7)
- [ ] Build a React component that renders an SVG pitch based on *relative* coordinates.
- [ ] Instead of hardcoding "Tait," the component will read `player_name` and `x,y` coordinates from the `visual_data` JSON field in the database.
- [ ] Implement the "Deep Link" logic: `openVideo(match.video_url + "?t=" + event.timestamp)`.

### Phase 3: Analytics & Narrative (Days 8-11)
- [ ] **TAS Gauges:** Build reusable UI components to show "Resilience" and "Synchrony" levels.
- [ ] **Multi-Match Navigation:** Create a sidebar that lets the user switch between different uploaded Matches/Teams.
- [ ] **Comparison Mode:** Allow the user to select one "Anomaly" event and one "Benchmark" event to show the contrast in TAS scores.

### Phase 4: Pilot Ready (Days 12-14)
- [ ] **Deployment:** Push to Vercel and create a "Demo" login.
- [ ] **Marketing Prep:** Ensure the dashboard looks "Elite" (Dark mode, high-contrast alerts).

## 💡 How to use this with Gemini in VS Code
When you start coding, give Gemini these specific prompts:
1. *"Gemini, generate a Supabase SQL schema for a sports analytics app that tracks teams, matches, and tactical events with JSON coordinates."*
2. *"Gemini, create a React SVG component of a football pitch that takes an array of player objects `{ name: string, x: number, y: number, color: string }` and renders them as circles."*
3. *"Gemini, write a utility function to format YouTube URLs to include a specific start time based on a database integer (seconds)."*

## ✅ Compliance & Standards
- All spelling: **UK English**.
- Data Handling: Minimalist and isolated by Team ID.
- Goal: Demonstrate **Explainable AI** (not just "what" happened, but "why" based on TAS/SynIQ).@AGENTS.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NorthbraeStudio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NorthbraeStudio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
