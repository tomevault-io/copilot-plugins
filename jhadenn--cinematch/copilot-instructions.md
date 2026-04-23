## cinematch

> **Due:** April 17, 2025

# CineMatch — Product Requirements Document

**Version:** 1.0  
**Due:** April 17, 2025  
**Team Size:** 5 members  

---

## Table of Contents

1. [Overview](#overview)
2. [Problem & Goals](#problem--goals)
3. [User Stories](#user-stories)
4. [Features & Requirements](#features--requirements)
5. [Technical Stack](#technical-stack)
6. [Application Structure](#application-structure)
7. [Vue Component Tree](#vue-component-tree)
8. [Express API Routes](#express-api-routes)
9. [Database Schema](#database-schema)
10. [Recommendation Engine](#recommendation-engine)
11. [Rubric Mapping](#rubric-mapping)
12. [Submission Checklist](#submission-checklist)

---

## Overview

CineMatch is a personalized movie discovery and recommendation web application. Users search and can save films to watchlists, while an intelligent recommendation engine learns their taste and surfaces movies they are likely to love. Rich D3-powered visualizations give users insight into their own viewing habits and preferences.

---

## Problem & Goals

Deciding what to watch is overwhelming — streaming platforms surface algorithmic content that favors their own catalogue. CineMatch puts the user in control: a personal watchlist and viewing history feed a transparent recommendation engine, and dashboards reveal patterns in their own taste over time.

**Goals:**
- Let users discover movies through search, filters, and smart recommendations
- Provide a personal watchlist and watch history system
- Visualize user taste with interactive D3 charts
- Demonstrate all course technologies in a cohesive, production-ready app

---

## User Stories

| As a...       | I want to...                                      | So that...                                      |
|---------------|---------------------------------------------------|-------------------------------------------------|
| Guest user    | Search and browse movies                          | I can explore content without an account        |
| Registered user | Save movies to my watchlist                    | I can track what I want to watch                |
| Registered user | Mark movies as watched                         | My history feeds the recommendation engine      |
| Registered user | Get personalized recommendations               | I discover movies tailored to my taste          |
| Registered user | View my taste dashboard                        | I understand my own viewing patterns            |
| Any user      | View full movie details with trailers            | I can decide if a movie interests me            |

---

## Features & Requirements

### 1. Search & Browse

- Full-text movie search via TMDB API
- Filter by genre, release year, language, and minimum rating
- Infinite scroll results with poster cards
- Trending and new release sections on the home page

### 2. Watchlist

- Add/remove movies to a personal watchlist
- Mark movies as watched (moves to history)
- Drag-and-drop reordering of watchlist items
- Shareable watchlist link via generated URL token

### 3. Movie Detail Page

- Full metadata: title, overview, genres, release date, runtime, language
- Embedded YouTube trailer via TMDB video endpoint
- Cast carousel with headshots from TMDB
- Similar movie suggestions pulled from TMDB `/movie/{id}/similar`
- Link to full cast and crew page

### 4. Recommendation Engine *(new technology)*
 
- Semantic similarity powered by OpenAI `text-embedding-3-small` model
- Each movie's overview + genres + director are embedded into a vector on first watch and cached in the DB
- A user profile vector is computed by averaging all watched movie embeddings
- Candidate movies (trending + popular from TMDB, not yet watched) are scored by cosine similarity against the profile vector
- Recommendation feed shown on home page and dedicated `/recommendations` page

### 5. Taste Dashboard

- **Genre breakdown** — D3 pie/donut chart of watched genres
- **Watch history timeline** — D3 line chart of movies watched per month
- **Decade heatmap** — D3 heatmap of movies by decade and genre
- **Top directors** — D3 horizontal bar chart of most-watched directors

### 6. Authentication

- Register and login with email and password
- Passwords hashed with `bcrypt`
- Sessions managed via JWT stored in `localStorage`
- Protected routes redirect unauthenticated users to login

---

## Technical Stack

| Layer         | Technology                          |
|---------------|-------------------------------------|
| Frontend      | Vue 3 (Composition API)             |
| Routing       | Vue Router                          |
| State         | Pinia                               |
| Styling       | Tailwind CSS                        |
| Charts        | D3.js                               |
| Backend       | Node.js + Express.js                |
| Database      | SQLite (via `better-sqlite3`)       |
| External API  | TMDB API (free tier)                |
| Auth          | JWT + bcrypt                        |
| New Tech      | OpenAI Embeddings API (`text-embedding-3-small`)  |

> **Why SQLite?** Zero setup, file-based, works out of the box on any machine for demo and grading purposes.

> **Why OpenAI embeddings?** `text-embedding-3-small` is cheap (fractions of a cent per request at this scale), production-grade, and semantically richer than hand-crafted genre vectors. Embedding vectors are generated once per movie and cached in the DB, so the API is rarely called after initial setup.

---
 
## Application Structure
 
```
cinematch/
├── client/                   # Vue 3 frontend
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── NavBar.vue
│   │   │   │   └── Footer.vue
│   │   │   ├── movie/
│   │   │   │   ├── MovieCard.vue
│   │   │   │   ├── MovieGrid.vue
│   │   │   │   ├── MovieDetail.vue
│   │   │   │   ├── CastCarousel.vue
│   │   │   │   └── TrailerEmbed.vue
│   │   │   ├── search/
│   │   │   │   ├── SearchBar.vue
│   │   │   │   └── FilterPanel.vue
│   │   │   ├── watchlist/
│   │   │   │   ├── WatchlistPanel.vue
│   │   │   │   └── WatchlistItem.vue
│   │   │   ├── dashboard/
│   │   │   │   ├── GenrePieChart.vue
│   │   │   │   ├── TimelineChart.vue
│   │   │   │   ├── DecadeHeatmap.vue
│   │   │   │   └── DirectorBarChart.vue
│   │   │   ├── recommendations/
│   │   │   │   └── RecoFeed.vue
│   │   │   └── auth/
│   │   │       ├── LoginForm.vue
│   │   │       └── RegisterForm.vue
│   │   ├── views/
│   │   │   ├── HomeView.vue
│   │   │   ├── SearchView.vue
│   │   │   ├── MovieView.vue
│   │   │   ├── WatchlistView.vue
│   │   │   ├── DashboardView.vue
│   │   │   ├── RecommendationsView.vue
│   │   │   ├── LoginView.vue
│   │   │   └── RegisterView.vue
│   │   ├── stores/
│   │   │   ├── auth.js
│   │   │   ├── watchlist.js
│   │   │   └── movies.js
│   │   ├── router/
│   │   │   └── index.js
│   │   ├── services/
│   │   │   ├── tmdb.js           # TMDB API calls
│   │   │   └── api.js            # Own backend API calls
│   │   └── App.vue
│   ├── index.html
│   └── vite.config.js
│
├── server/                   # Express.js backend
│   ├── routes/
│   │   ├── auth.js
│   │   ├── watchlist.js
│   │   ├── history.js
│   │   └── recommendations.js
│   ├── middleware/
│   │   └── auth.js            # JWT verification middleware
│   ├── db/
│   │   ├── schema.sql
│   │   └── database.js
│   ├── services/
│   │   └── recommender.js     # OpenAI embedding-based recommendation engine
│   └── index.js
│
├── group_members.html
├── contributions.txt
├── intro.mp4
├── .env.example
├── package.json
└── README.md
```
 
---
 
## Vue Component Tree
 
```
App.vue
├── NavBar.vue
├── router-view
│   ├── HomeView.vue
│   │   ├── SearchBar.vue
│   │   ├── MovieGrid.vue (trending)
│   │   │   └── MovieCard.vue (×n)
│   │   └── RecoFeed.vue (if logged in)
│   │       └── MovieCard.vue (×n)
│   │
│   ├── SearchView.vue
│   │   ├── SearchBar.vue
│   │   ├── FilterPanel.vue
│   │   └── MovieGrid.vue (results, infinite scroll)
│   │       └── MovieCard.vue (×n)
│   │
│   ├── MovieView.vue
│   │   ├── TrailerEmbed.vue
│   │   ├── CastCarousel.vue
│   │   └── MovieGrid.vue (similar movies)
│   │
│   ├── WatchlistView.vue
│   │   └── WatchlistItem.vue (×n, drag-and-drop)
│   │
│   ├── DashboardView.vue
│   │   ├── GenrePieChart.vue       (D3)
│   │   ├── TimelineChart.vue       (D3)
│   │   ├── DecadeHeatmap.vue       (D3)
│   │   └── DirectorBarChart.vue    (D3)
│   │
│   ├── RecommendationsView.vue
│   │   └── RecoFeed.vue
│   │       └── MovieCard.vue (×n)
│   │
│   ├── LoginView.vue
│   │   └── LoginForm.vue
│   │
│   └── RegisterView.vue
│       └── RegisterForm.vue
│
└── Footer.vue
```
 
---


---

## Express API Routes

### Auth — `/api/auth`

| Method | Route       | Description              | Auth required |
|--------|-------------|--------------------------|---------------|
| POST   | `/register` | Create new account       | No            |
| POST   | `/login`    | Login, returns JWT       | No            |
| GET    | `/me`       | Get current user profile | Yes           |

### Watchlist — `/api/watchlist`

| Method | Route    | Description                        | Auth required |
|--------|----------|------------------------------------|---------------|
| GET    | `/`      | Get user's watchlist               | Yes           |
| POST   | `/`      | Add movie to watchlist             | Yes           |
| DELETE | `/:id`   | Remove movie from watchlist        | Yes           |
| PATCH  | `/order` | Update watchlist item order        | Yes           |
| GET    | `/share/:token` | Get shared watchlist (public) | No        |

### Watch History — `/api/history`

| Method | Route   | Description                      | Auth required |
|--------|---------|----------------------------------|---------------|
| GET    | `/`     | Get user's full watch history    | Yes           |
| POST   | `/`     | Mark a movie as watched          | Yes           |
| DELETE | `/:id`  | Remove from history              | Yes           |

### Recommendations — `/api/recommendations`

| Method | Route | Description                                      | Auth required |
|--------|-------|--------------------------------------------------|---------------|
| GET    | `/`   | Get personalized recommendations for current user | Yes           |

---

## Database Schema

```sql
-- Users
CREATE TABLE users (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  email       TEXT UNIQUE NOT NULL,
  password    TEXT NOT NULL,            -- bcrypt hash
  username    TEXT NOT NULL,
  created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Watchlist
CREATE TABLE watchlist (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id     INTEGER NOT NULL REFERENCES users(id),
  tmdb_id     INTEGER NOT NULL,
  title       TEXT NOT NULL,
  poster_path TEXT,
  position    INTEGER NOT NULL DEFAULT 0,
  added_at    DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, tmdb_id)
);

-- Watch history
CREATE TABLE watch_history (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id     INTEGER NOT NULL REFERENCES users(id),
  tmdb_id     INTEGER NOT NULL,
  title       TEXT NOT NULL,
  poster_path TEXT,
  genres      TEXT NOT NULL,            -- JSON array: ["Action", "Drama"]
  director    TEXT,
  release_year INTEGER,
  watched_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, tmdb_id)
);

-- Shared watchlist tokens
CREATE TABLE watchlist_shares (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id     INTEGER NOT NULL REFERENCES users(id),
  token       TEXT UNIQUE NOT NULL,
  created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

---

## Recommendation Engine

**Package:** `ml-matrix` (npm)  
**Approach:** Content-based filtering using cosine similarity

### How it works

1. **Build a feature vector** for each movie in the user's watch history using:
   - Genre weights (one-hot encoded across ~19 TMDB genres)
   - Decade (normalized 0–1)
   - Director match bonus
   
2. **Build a user profile vector** by averaging all watched movie vectors

3. **Score candidate movies** (trending + popular from TMDB not yet watched) by computing cosine similarity against the user profile vector

4. **Return top N results** sorted by similarity score

### Implementation sketch (`server/services/recommender.js`)

```js
const { Matrix } = require('ml-matrix');

const GENRES = [
  'Action','Adventure','Animation','Comedy','Crime',
  'Documentary','Drama','Family','Fantasy','History',
  'Horror','Music','Mystery','Romance','Science Fiction',
  'Thriller','War','Western','TV Movie'
];

const DECADE_MIN = 1920;
const DECADE_MAX = 2030;

function buildVector(movie) {
  const genreVec = GENRES.map(g => movie.genres.includes(g) ? 1 : 0);
  const decade = (movie.release_year - DECADE_MIN) / (DECADE_MAX - DECADE_MIN);
  return [...genreVec, decade];
}

function cosineSimilarity(a, b) {
  const dot = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const magA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const magB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dot / (magA * magB);
}

function getRecommendations(watchedMovies, candidates) {
  if (watchedMovies.length === 0) return candidates.slice(0, 10);

  const watchedVectors = watchedMovies.map(buildVector);
  const profileMatrix = new Matrix(watchedVectors);
  const profile = profileMatrix.mean('column'); // average vector

  return candidates
    .map(movie => ({
      ...movie,
      score: cosineSimilarity(profile, buildVector(movie))
    }))
    .sort((a, b) => b.score - a.score)
    .slice(0, 20);
}

module.exports = { getRecommendations };
```

---

## Rubric Mapping

| Requirement          | Implementation                                                                 | Marks |
|----------------------|--------------------------------------------------------------------------------|-------|
| SVG & HTML           | Custom SVG logo, SVG icons for genre badges, watchlist empty state illustration | 1.0   |
| CSS & frameworks     | Tailwind CSS utility classes, custom CSS transitions for card hover states      | 1.0   |
| JavaScript, jQuery, D3 | D3 genre pie, timeline line chart, decade heatmap, director bar chart        | 1.5   |
| Dynamic DOM          | Live search filtering, infinite scroll results, drag-and-drop watchlist order  | 1.5   |
| AJAX & web services  | TMDB API (search, details, trailers, similar); own REST API for user data      | 1.0   |
| Node / Express       | Express REST API: auth, watchlist, history, recommendations endpoints          | 1.5   |
| Vue framework        | Composition API components, Vue Router, Pinia stores, reactive data binding    | 2.5   |
| New technology       | Content-based ML recommendation engine using `ml-matrix` npm package           | 1.0   |
| Intro video          | 3–5 min screen recording demonstrating all features and technologies            | 4.0   |
| **Total**            |                                                                                | **10.0** |


## Submission Checklist
- [ ] No hardcoded API keys (use `.env` with `.env.example` provided)
- [ ] App runs with `npm install` + `npm run dev` from project root
- [ ] `group_members.html` — table with all member names and Banner IDs
- [ ] `contributions.txt` — each member's specific contributions listed
- [ ] `intro.mp4` — 3–5 min promo video demonstrating all features (and new technology)
- [ ] README with clear setup steps (clone → install → add `.env` → run)

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhadenn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
