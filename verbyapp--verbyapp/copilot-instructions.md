## verbyapp

> This document provides guidelines for agents working on the Verby codebase.

# Agent Guidelines for Verby

This document provides guidelines for agents working on the Verby codebase.

---

## Project Overview

Verby is a French verb conjugation learning app built with React, Vite, Tailwind CSS, Firebase, and React Router v7.

**Game Modes:**
- **Blitz**: 60-second timed game with ELO rating system (+2 correct, -2 wrong, +4 for streaks after 2 consecutive)
- **Verby Streak**: Endless mode - one mistake ends the game. Tracks best streak, no rating
- **Zen Mode**: Practice at your own pace with customizable modes/tenses. Tracks correct/wrong ratio, no rating
- **Ranked Duels**: (Coming Soon) Head-to-head multiplayer
- **Daily Quest**: (Coming Soon) Daily challenges

---

## Commands

### Development
```sh
npm run dev          # Start development server at localhost:5173
npm run preview      # Preview production build
```

### Build & Lint
```sh
npm run build        # Production build to /dist
npm run lint         # Run ESLint on all files
```

### Single File Linting
```sh
npx eslint src/pages/VerbyLanding.jsx    # Lint specific file
npx eslint src/**/*.jsx --fix             # Lint with auto-fix
```

---

## Code Style Guidelines

### General Principles
- Use functional components with hooks (no class components)
- Prefer `const` over `let` for variable declarations
- Use named exports for utilities, default exports for page components
- Always run `npm run lint` before committing

### File Organization
```
src/
├── App.jsx                 # Routes configuration with ProtectedRoute
├── main.jsx               # Entry point
├── context/
│   └── AuthContext.jsx    # Firebase auth context
├── lib/
│   └── firebase.js       # Firebase initialization
├── components/
│   └── MainNavbar.jsx     # Main navigation with mobile menu
├── pages/
│   ├── arena/             # Game modes (Blitz, VerbyStreak, ZenMode, Arena)
│   ├── community/         # Leaderboards (blitz, duels, mastery, daily)
│   ├── profile/          # Profile pages (profile, PublicProfile, EditProfile)
│   ├── tools/             # Tools (search, etc.)
│   └── admin/            # Admin dashboard
└── assets/               # Audio files (correct.mp3, wrong.mp3, 10.mp3, done.mp3)
```

### Import Order
1. React hooks (`useState`, `useEffect`, `useCallback`, `useRef`, etc.)
2. React Router (`Link`, `useNavigate`, `useParams`)
3. Firebase (`ref`, `get`, `update`, `onValue`, `signInWithPopup`, etc.)
4. Third-party libraries (lucide-react icons, sonner, recharts)
5. Components/utils
6. Assets

```jsx
import { useState, useEffect, useCallback } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { ref, update, get } from 'firebase/database';
import { database } from '../../lib/firebase';
import { toast } from 'sonner';
import { Zap, ArrowLeft } from 'lucide-react';
import MainNavbar from '../../components/MainNavbar';
import correctSound from '../../assets/correct.mp3';
```

### Naming Conventions
- Components: PascalCase (`VerbyLanding`, `MainNavbar`, `CommunityBlitz`)
- Functions/Variables: camelCase (`handleAnswer`, `fetchVerb`, `isLoading`)
- Files: PascalCase for components (`Blitz.jsx`), camelCase for utils (`utils.js`)
- Routes: kebab-case (`/arena/blitz`, `/profile/:id`)

### Component Structure
```jsx
// Define sub-components at the bottom of the file
const GameCard = ({ title, stats }) => (
  <div className="p-4 bg-white rounded-lg">
    <h3>{title}</h3>
    <p>{stats}</p>
  </div>
);

// Main component exported at the bottom
const Arena = () => {
  // hooks first
  const { user } = useAuth();
  
  // then state and refs
  
  // then useEffects
  
  // then callbacks
  
  // then render
  return (
    <div>
      <GameCard title="Blitz" stats="1200 ELO" />
    </div>
  );
};

export default Arena;
```

### Tailwind CSS
- Use the project's custom color variables: `text-[#EB3514]`, `bg-[#F0EFEB]`, `border-[#DEDDDA]`
- Font: `font-mono` for UI text, `font-sans` for headings
- Responsive: `md:` prefix for desktop styles, mobile-first approach

### Error Handling
```jsx
// Async operations
try {
  const response = await fetch(url);
  if (!response.ok) {
    throw new Error('Failed to fetch');
  }
  const data = await response.json();
  setData(data);
} catch (error) {
  console.error('Error:', error);
  toast.error('Failed to load data');
}
```

### Firebase Data Structures

**User Profile** (`/users/{uid}/profile/`):
```js
{
  displayName: "Username",
  email: "user@example.com",
  photoURL: "https://...",
  bio: "Optional bio",
  country: "dz",
  languages: { default: "en" },  // Note: stored as object, NOT array
  role: "admin",  // or "user"
  createdAt: timestamp,
  lastLogin: timestamp
}
```

**Game Stats** (`/users/{uid}/stats/`):
```js
{
  blitz: { rating: 1200, games: 10, wins: 5 },
  streak: { bestStreak: 15, lastStreak: 8, games: 3 },
  zen: { correct: 50, wrong: 5 }
}
```

**Game History** (`/users/{uid}/gameHistory/{mode}/{gameId}`):
```js
{
  timestamp: Date.now(),
  correct: 20,
  total: 25,
  score: 85,
  // or for streak:
  streak: 15,
  bestStreak: 15
}
```

**Rating History** (`/users/{uid}/ratingHistory/{date}`):
```js
{
  blitz: 1200,
  duels: 1200,
  streak: 1200  // Note: stored as number, not used for ranking
}
// Date format: "2026-03-26" (YYYY-MM-DD)
```

---

## API Endpoints

VerbyBack API (https://verby-back.vercel.app/api):

```js
// Get random verb with conjugations
GET /random/{mode}  // modes: indicatif, subjonctif, conditionnel, imperatif
// Returns: { verb, mode, tenses: { "Présent": [...], "Passé composé": [...] } }

// Search verbs by prefix
GET /search/{prefix}  // e.g., /search/m
// Returns: ["manger", "manquer", "marcher", ...]

// Get full conjugation
GET /conjugate/{verb}  // e.g., /conjugate/manger
// Returns: { verb, conjugations: { Indicatif: {...}, Subjonctif: {...} } }
```

---

## Routing

Routes are configured in `App.jsx` using React Router v7:

```jsx
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';

const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return null;
  if (!user) return <Navigate to="/login" replace />;
  return children;
};

// Routes
<Route path="/" element={<VerbyLanding />} />
<Route path="/login" element={<Login />} />
<Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />
<Route path="/profile/:id" element={<PublicProfile />} />
<Route path="/arena" element={<ProtectedRoute><Arena /></ProtectedRoute>} />
<Route path="/arena/blitz" element={<ProtectedRoute><Blitz /></ProtectedRoute>} />
```

---

## Important Patterns

### Game Timer (Blitz)
```jsx
const timerRef = useRef(null);
const scoreRef = useRef(0);  // Use refs for values updated in timer intervals

useEffect(() => {
  if (gameState === 'playing') {
    timerRef.current = setInterval(() => {
      // Update refs, not state directly for performance
      scoreRef.current += 2;
      setScore(scoreRef.current);
    }, 1000);
  }
  return () => {
    if (timerRef.current) clearInterval(timerRef.current);
  };
}, [gameState]);
```

### Firebase Real-time Updates
```jsx
useEffect(() => {
  const statsRef = ref(database, `users/${user.uid}/stats`);
  const unsubscribe = onValue(statsRef, (snapshot) => {
    if (snapshot.exists()) {
      setStats(snapshot.val());
    }
  });
  return () => unsubscribe();
}, [user]);
```

### Pronoun Normalization (for apostrophe verbs)
```jsx
const extractPersonFromConjugation = (conjugation) => {
  // Handle "j'", "n'", "l'", "t'", "s'", "qu'"
  if (conjugation.startsWith("j'")) return { person: 'je', verbPart: conjugation.slice(2) };
  if (conjugation.startsWith("n'")) return { person: 'nous', verbPart: conjugation.slice(2) };
  // ... etc
};
```

---

## Dependencies

Key dependencies:
- `react` / `react-dom`: UI framework (v19)
- `react-router-dom`: Routing (v7)
- `firebase`: Authentication and database (Realtime Database)
- `tailwindcss`: Styling
- `lucide-react`: Icons
- `sonner`: Toast notifications
- `recharts`: Charts for rating graphs
- `prop-types`: Prop validation for ProtectedRoute

---

## Linting Rules

Always fix ESLint errors before committing. Common issues:
- `no-unused-vars`: Remove unused imports/variables
- `react-hooks/exhaustive-deps`: Add missing dependencies to useEffect/useCallback
- `react/no-unescaped-entities`: Use `&apos;` for apostrophes in JSX
- `prop-types`: Add PropTypes for custom components with props

---
> Source: [Verbyapp/Verbyapp](https://github.com/Verbyapp/Verbyapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
