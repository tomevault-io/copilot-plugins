## inventario-activos-isc

> **SGIAC-ISC** - Sistema de Gestión de Inventarios de Activos y Consumibles

# AGENTS.md - Guidelines for SGIAC-ISC Development

## Project Overview

**SGIAC-ISC** - Sistema de Gestión de Inventarios de Activos y Consumibles  
Stack: Express.js (Backend) + Vanilla JS (Frontend) + Supabase (Database)

---

## 1. Build & Development Commands

### Backend
```bash
cd backend
npm install          # Install dependencies
npm run dev          # Start server with nodemon (port 3000)
```

### Frontend
- Served statically by backend at `http://localhost:3000`
- No build process required (vanilla JS + HTML)

### Testing
- **No test framework configured** - Do not add tests unless explicitly requested
- If adding tests later, use Jest for backend

---

## 2. Code Style Guidelines

### General Conventions

- **Language**: Spanish for UI text, English for code identifiers
- **Commits**: Spanish (imperative mood)
- **Line ending**: LF (Unix-style)

### Backend (Express.js)

#### Structure
- Routes in `/backend/src/*.js`
- Main server: `/backend/server.js`
- Use Express Router for each resource

#### Naming Conventions
```javascript
// Routes: nouns plural (REST API)
router.get("/", ...)      // List
router.get("/:id", ...)   // Get by ID
router.post("/", ...)     // Create
router.put("/:id", ...)   // Update
router.delete("/:id", ...)// Delete

// Files: kebab-case (snake_case for DB tables)
assets.js, users.js, consumibles.js  // routes
assets, users, consumibles           // DB tables
```

#### Error Handling
```javascript
// Always return JSON with status code
if (error) return res.status(500).json(error);
if (!id) return res.status(400).json({ message: "ID requerido" });
```

#### Database (Supabase)
- Use `supabase` client from `./src/supabase`
- Query syntax:
```javascript
const { data, error } = await supabase
  .from("assets")
  .select("*, categories(name)")
  .eq("id", req.params.id);
if (error) return res.status(500).json(error);
```

### Frontend (Vanilla JS)

#### File Structure
- `/frontend/src/js/` - JavaScript files
- `/frontend/src/*.html` - HTML pages

#### Naming Conventions
```javascript
// Functions: camelCase
function getUser() { ... }
function requireAuth() { ... }

// Constants: UPPER_SNAKE_CASE
const API_BASE = "/api";
const PAGE_PERMISSIONS = { ... };

// DOM elements: descriptive, prefixed
const userImg = document.querySelector(".user img");
const logoutBtn = document.getElementById("logoutBtn");
```

#### JavaScript Patterns
```javascript
// API calls via fetch
async function fetchAssets() {
  const res = await fetch("/api/assets");
  return res.json();
}

// DOM ready
document.addEventListener("DOMContentLoaded", initPage);

// Inline styles use template literals
element.style.cssText = `
  position:fixed;top:0;left:0;
  background:rgba(15,23,42,0.6);
`;
```

#### HTML Structure
```html
<!-- Use semantic tags -->
<nav class="sidebar">...</nav>
<main class="main">...</main>
<header class="header">...</header>

<!-- Icons from Font Awesome -->
<i class="fas fa-box"></i>
<i class="fas fa-user"></i>

<!-- Data attributes for JS hooks -->
<button data-action="delete" data-id="5">Eliminar</button>
```

### CSS Guidelines

- Use CSS variables for colors:
```css
:root {
  --primary: #4f46e5;
  --success: #10b981;
  --warning: #f59e0b;
  --danger: #ef4444;
}
```

- Follow BEM-lite for classes:
```css
.card { ... }
.card-info { ... }
.card-icon { ... }
```

- Responsive: mobile-first with media queries
```css
@media (max-width: 768px) { ... }
```

---

## 3. Database Schema (Supabase)

Tables use **snake_case**:
- `assets`, `users`, `consumibles`, `requests`, `reservations`, `logs`, `categories`, `labs`
- Foreign keys: `${table}_id` (e.g., `category_id`, `user_id`)

---

## 4. API Endpoints

| Resource | Endpoints |
|----------|-----------|
| Auth | `/api/auth/*` |
| Assets | `/api/assets/*` |
| Categories | `/api/categories/*` |
| Users | `/api/users/*` |
| Consumibles | `/api/consumibles/*` |
| Requests | `/api/requests/*` |
| Reservations | `/api/reservations/*` |
| Stats | `/api/stats/*` |
| Labs | `/api/labs` |

---

## 5. Important Notes

- **No tests configured** - Ask before adding test frameworks
- **No lint configured** - Follow existing code style
- **Supabase credentials** in `.env` file (do not commit)
- **Frontend assets** in `/frontend/public/`
- **Role hierarchy**: administrador > docente > alumno

---

## 6. Add-On Commands (If Needed)

```bash
# Add Jest for backend testing
npm install --save-dev jest
npm test                   # Run tests (configure jest.config.js first)

# Add ESLint
npm install --save-dev eslint
npx eslint backend/src/    # Lint backend
```

---
> Source: [Chiliswilis/Inventario-Activos-ISC](https://github.com/Chiliswilis/Inventario-Activos-ISC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
