## life-dots

> This document (`gemini.md`) serves as the primary context for any AI assistant working on the **Life in Dots** project. It outlines the project's architecture, technologies, conventions, and existing features.

# Life In Dots - AI Assistant Context Document

This document (`gemini.md`) serves as the primary context for any AI assistant working on the **Life in Dots** project. It outlines the project's architecture, technologies, conventions, and existing features. 

## 📖 Project Overview
**Life In Dots** is a minimalist *memento mori* dashboard designed to track user lifespan as a grid of dots. Each dot typically represents an increment of time (e.g., a month), providing visual perspective on time and acting as a canvas to document one's life.

## 🛠 Tech Stack
The project is split into a separate frontend and backend, using modern JS tooling.
### Frontend
- **Framework:** React 19 + Vite (built with ES Modules).
- **Styling:** Tailwind CSS (v4) + Vanilla CSS (`index.css`) for core utility handling.
- **Animations:** Framer Motion (for heartbeat, view/modal transitions).
- **Rich Text / Journaling:** Tiptap (Headless rich text editor with extensions).
- **Icons:** Lucide-React.
- **Other utilities:** `html-to-image` for layout export functionality.

### Backend
- **Framework:** Node.js (v22+) + Express.js 5.
- **Database/Auth:** Supabase JS Client (using Postgres backend on Supabase).
- **Validation:** Zod (for request validation and typing).

## 📁 Directory Structure
```
/
├── frontend/
│   ├── index.html        (Main Vite entry)
│   ├── package.json      (React 19, Tailwind v4, Vite)
│   ├── src/
│   │   ├── main.jsx      (React DOM render)
│   │   ├── App.jsx       (Core application router/state logic)
│   │   ├── index.css     (Global CSS)
│   │   ├── components/   (Reusable UI elements, Journal/ToDo views)
│   │   ├── hooks/        (Custom React hooks, API fetching wrappers)
│   │   ├── utils/        (Utility functions: date maths, Supabase clients)
│   │   └── assets/       (Static assets)
│
└── backend/
    ├── package.json      (Express 5, Zod, Supabase)
    ├── src/
    │   ├── index.js      (Express server entry point)
    │   ├── controllers/  (Route logic handlers)
    │   ├── routes/       (Express REST API routes)
    │   ├── services/     (Business logic and Database interaction)
    │   ├── middleware/   (Auth checks, error boundaries)
    │   └── lib/          (Shared backend utils: Supabase instance)
```

## 🔑 Key Features
1. **Interactive Timeline Grid:** Years/Months/Days rendered as dots. Users can customize birth date and expected lifespan.
2. **Journaling Entries:** Associated with specific time increments. Uses a rich text editor (Tiptap). Users can manually save/tick entries.
3. **Dedicated To-Do System:** Recently decoupled from Journal views, given a dedicated full-page list view and individual components.
4. **Customization & Settings:**
   - **Themes:** Light / Dark modes.
   - **Custom Dot Colors:** Users can pick a custom default dot color. Reverting resets back to theme default.
   - **Heartbeat Animation:** Toggleable in settings.

## 🏗 Coding Guidelines & Conventions
- **Absolute Paths & Imports:** Keep imports clean and verify you are referencing the correct `backend` or `frontend` dependencies. Both use `"type": "module"`.
- **Component Design (Frontend):** Focus on dynamic, responsive layouts. Tailwind CSS classes are preferred, alongside headless component structuring. The UI should always feel premium, fluid, and animated.
- **State Management:** Keep modals and overlays decoupled where possible. Moving towards full-page views over constrained modals when data is large (e.g. All Journals or All To-Dos).
- **Data Fetching (Frontend):** Ensure data syncs correctly and handles failures smoothly. Always use proper error catching.
- **Validation (Backend):** Every request must go through Zod validation schema before hitting the controllers.
- **Supabase Interaction:** Supabase does the heavy lifting for DB queries. The backend acts as a secure proxy to fetch, filter, and validate data before sending it to the frontend.

## 💡 Reminders for AI (You)
- Always check `.env` configuration first when debugging auth or network issues. (E.g. missing Anon Key or Service Role key).
- When altering UI flows, remember the user has explicitly requested to move from dense modals to full-page, breathable views.
- Ensure any database schema changes are reflected in Zod schemas on the backend and in any static type/prop declarations on the frontend.
- Provide descriptive terminal instructions to run `npm run dev` in both directories for local testing. Wait on concurrent port mapping (Vite: 5173, Express backend: varies).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Avi161)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Avi161)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
