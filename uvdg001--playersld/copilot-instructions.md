## playersld

> **Players LD** is a professional-grade football team management application. The project is currently in a transition phase, refactoring a monolithic "V1" (found in the root directory) into a modular, high-performance "V2" (found in the `/v2` directory) with a premium aesthetic.

# ⚽ Players LD - Instructional Context

## 🎯 Project Overview
**Players LD** is a professional-grade football team management application. The project is currently in a transition phase, refactoring a monolithic "V1" (found in the root directory) into a modular, high-performance "V2" (found in the `/v2` directory) with a premium aesthetic.

- **Purpose:** Manage match fixtures, player statistics, treasury/payments, team generation via AI, and social features for football teams.
- **Core Architecture:** React (TypeScript) + Vite + Firebase (Firestore).
- **Design Philosophy:** "Premium & Mobile First" - Ultra-rounded corners (`rounded-[2.5rem]`), heavy typography, and high-impact visual feedback.

---

## 🏗️ Project Structure & Strategy

### 🧪 The Sandbox Strategy (`/v2`)
- **Root Directory:** Contains the production-ready V1. **Do not modify directly** for experimental features.
- **`/v2` Directory:** The development laboratory. This is where the refactoring and modularization take place.
- **Modularization Goal:** Deconstruct the massive `App.tsx` from V1 into independent pages and components within `/v2/src/pages` and `/v2/src/components`.

### 📂 Key Directories
- `components/`: UI components (V1).
- `services/`: Data logic, Firebase integration, and Gemini AI services.
- `contexts/`: Global state management (Language, Toast).
- `v2/src/pages/`: Modularized pages for the new version (Tournaments, Matches, Roster, etc.).
- `hooks/`: Custom React hooks for local storage and notifications.

---

## 💎 Development & Design Conventions

### 🎨 Visual Identity (Strict Rules)
- **Palette:** Prefer **Indigo**, **Emerald**, and **Slate**.
- **PROHIBITED:** Never mix **Blue** with **Yellow**.
- **Styling:** Use Tailwind CSS. Focus on `rounded-[2.5rem]` or `rounded-full` for a modern, "app-like" feel.
- **Typography:** Black/Heavy weights, uppercase, and italicized accents for headings.

### 📊 Data Integrity
Maintain all historical data fields for players:
- Shirt Number (#), Position, Preferred Foot.
- DNI, Phone Number, Birthday (used for age calculation and birthday cards).
- Stats: Goals, Assists, MVP counts, etc.

### 🛠️ Technical Stack
- **Framework:** React 18 (TypeScript).
- **Build Tool:** Vite.
- **Database:** Firebase Firestore (Compat Layer currently in use).
- **AI:** Google Generative AI (`@google/genai`) for intelligent team balancing.
- **Animations:** Framer Motion (`motion`).

---

## 🚀 Key Commands

### Development
- `npm run dev`: Start the V1 development server.
- `cd v2 && npm install && npm run dev`: Start the V2 development laboratory.

### Build & Deploy
- `npm run build`: Build the V1 project.
- `npm run deploy`: Build and deploy V1 to Firebase.
- `cd v2 && npm run build && firebase hosting:channel:deploy v2-premium`: Deploy V2 to a temporary preview channel.

---

## 💡 Future Roadmap
1. Complete the "transplant" of all V1 features into the modular V2 structure.
2. Implement "Standby/Lock" logic for player rosters.
3. Enhance statistics visualization with premium charts.
4. Deepen Gemini AI integration for tactical analysis.

---
> Source: [uvdg001/PlayersLD](https://github.com/uvdg001/PlayersLD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
