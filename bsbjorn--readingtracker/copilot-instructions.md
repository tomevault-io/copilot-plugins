## readingtracker

> This file contains essential context for AI assistants working on the Reading Tracker project.

# AGENTS.md - AI Assistant Guide

This file contains essential context for AI assistants working on the Reading Tracker project.

## Project Overview

**Name:** Reading Tracker  
**Type:** Full-stack web application for tracking personal reading habits  
**Architecture:** Monorepo with separate backend and frontend directories  
**Status:** Fully functional, actively maintained

## Technology Stack

### Backend (`/backend`)
- **Runtime:** Node.js 18+
- **Framework:** Express.js
- **Database:** PostgreSQL 16 (running in Podman/Docker container)
- **ORM:** Raw SQL queries with `pg` library
- **External APIs:** Google Books API
- **Key Dependencies:**
  - `express` - Web framework
  - `pg` - PostgreSQL client
  - `axios` - HTTP client for Google Books API
  - `cors` - CORS middleware
  - `dotenv` - Environment variables

### Frontend (`/frontend`)
- **Framework:** Vue 3 with Composition API (`<script setup>`)
- **Build Tool:** Vite
- **Styling:** Tailwind CSS v4 (using `@tailwindcss/vite` plugin, NOT PostCSS)
- **Routing:** Vue Router
- **State Management:** Pinia (configured but minimal usage)
- **Charts:** Chart.js + vue-chartjs
- **HTTP Client:** Axios
- **Key Pattern:** All components use Composition API with `<script setup>` syntax

### Database
- **System:** PostgreSQL 16
- **Container:** Managed via `./db.sh` helper script
- **Connection:** Default localhost:5432
- **Management:** Podman (Docker compatible)

## Directory Structure

```
ReadingTracker/
├── backend/                    # Express API server
│   ├── config/db.js           # Database connection pool
│   ├── routes/
│   │   ├── books.js           # Book CRUD + Google Books search
│   │   └── stats.js           # Statistics endpoints
│   ├── services/
│   │   └── bookApiService.js  # Google Books API integration
│   ├── server.js              # Main entry point
│   ├── .env                   # Environment config (NOT in git)
│   └── package.json
│
├── frontend/                   # Vue 3 SPA
│   ├── src/
│   │   ├── assets/
│   │   │   └── main.css       # Tailwind CSS config with @theme
│   │   ├── components/
│   │   │   └── MonthlyPagesChart.vue
│   │   ├── router/index.js
│   │   ├── services/api.js    # Axios API client
│   │   ├── stores/            # Pinia stores (minimal usage)
│   │   ├── utils/
│   │   │   └── isbnDetector.js
│   │   └── views/
│   │       ├── Dashboard.vue
│   │       ├── BooksView.vue
│   │       └── AddBookView.vue
│   ├── vite.config.js         # Uses @tailwindcss/vite plugin
│   ├── .env                   # Environment config (NOT in git)
│   └── package.json
│
├── migrations/                 # Database migration files
├── docs/                      # Original architecture docs
├── init-db.sql                # Database schema
├── db.sh                      # Database helper script
└── docker-compose.yml         # PostgreSQL container config
```

## Database Schema

### `books` Table
```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255),
    pages INTEGER,
    genres TEXT[],              -- PostgreSQL array type
    isbn VARCHAR(20),
    cover_url VARCHAR(500),
    source VARCHAR(20),         -- 'manual' or 'google_books'
    start_date DATE,            -- When user started reading
    end_date DATE,              -- When user finished reading
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Indexes
- `idx_books_start_date` on `start_date`
- `idx_books_end_date` on `end_date`

## Important Implementation Details

### 1. Tailwind CSS v4 Configuration
- **DO NOT** create `tailwind.config.js` or `postcss.config.js`
- Configuration is in `frontend/src/assets/main.css` using `@theme` directive
- Uses `@tailwindcss/vite` plugin in `vite.config.js`
- Import: `@import "tailwindcss";` (NOT `@tailwind base; @tailwind components;`)

### 2. Vue Composition API
- **ALL** components use `<script setup>` syntax
- Use `ref()` for primitive reactive values
- Use `reactive()` for objects
- Import lifecycle hooks: `onMounted`, `onUnmounted`, etc.
- Router: `const router = useRouter()` then `router.push()`
- Route params: `const route = useRoute()` then `route.params.id`

### 3. Date Handling
- Database stores dates as PostgreSQL `DATE` type
- Frontend displays in Norwegian format: dd.mm.yyyy
- HTML5 date inputs for user input
- Format function: `toLocaleDateString('nb-NO', { day: '2-digit', month: '2-digit', year: 'numeric' })`

### 4. Google Books API
- Service: `backend/services/bookApiService.js`
- No API key required (rate-limited)
- Optional API key via `GOOGLE_BOOKS_API_KEY` env var
- Search endpoint: `/api/books/search?q=query`
- ISBN endpoint: `/api/books/isbn/:isbn`

### 5. PostgreSQL Arrays
- Genres stored as TEXT[] array type
- Query with `UNNEST()` for statistics
- Insert/Update: `ARRAY['Genre1', 'Genre2']`
- Frontend: Comma-separated string converted to array

## Environment Configuration

### Development Mode (default)
```
Frontend: http://localhost:5173 (Vite dev server)
Backend:  http://localhost:3000
Database: localhost:5432
```

**Backend `.env`:**
```env
DATABASE_URL=postgresql://reading_user:reading_pass@localhost:5432/reading_tracker
PORT=3000
CORS_ORIGIN=http://localhost:5173
NODE_ENV=development
GOOGLE_BOOKS_API_KEY=
```

**Frontend `.env`:**
```env
VITE_API_URL=http://localhost:3000/api
```

### Docker Mode
```
Frontend: http://localhost:8080 (nginx)
Backend:  http://localhost:3000
Database: postgres:5432 (Docker network)
```

## Common Development Tasks

### Adding a New API Endpoint
1. Create route handler in `backend/routes/`
2. Register route in `backend/server.js`
3. Add API method to `frontend/src/services/api.js`
4. Use in Vue components

### Adding a New Vue Component
1. Create `.vue` file in `frontend/src/components/` or `frontend/src/views/`
2. Use `<script setup>` syntax
3. Import `ref`, `reactive`, `computed` as needed
4. Use Tailwind utility classes (no `<style>` sections)
5. If page component, add route to `frontend/src/router/index.js`

### Modifying Database Schema
1. Create migration SQL file in `migrations/`
2. Apply with: `podman exec -i reading-tracker-db psql -U reading_user -d reading_tracker < migrations/filename.sql`
3. Update relevant backend routes/queries
4. Update frontend forms/displays

### Adding Statistics
1. Add SQL query to `backend/routes/stats.js`
2. Use proper date functions: `DATE_TRUNC()`, `EXTRACT()`
3. For genres: Use `UNNEST(genres)` to query array
4. Add API method to `frontend/src/services/api.js`
5. Display in `Dashboard.vue` or new component

## API Endpoints Reference

### Books
- `GET /api/books` - List all books from database
- `GET /api/books/:id` - Get single book
- `POST /api/books` - Create book
- `PUT /api/books/:id` - Update book
- `DELETE /api/books/:id` - Delete book
- `GET /api/books/search?q=query&maxResults=10` - Search Google Books
- `GET /api/books/isbn/:isbn` - Get book by ISBN from Google Books

### Statistics
- `GET /api/stats/dashboard` - Dashboard overview (counts, averages, top genres)
- `GET /api/stats/monthly` - Last 12 months of reading data
- `GET /api/stats/genres` - All genres with book counts

## Key Files to Understand

1. **`backend/server.js`** - Express app setup, middleware, route registration
2. **`backend/config/db.js`** - PostgreSQL connection pool
3. **`backend/routes/books.js`** - All book-related endpoints
4. **`backend/routes/stats.js`** - Statistics queries
5. **`backend/services/bookApiService.js`** - Google Books API integration
6. **`frontend/src/services/api.js`** - Centralized API client
7. **`frontend/src/views/AddBookView.vue`** - Most complex component (search + form)
8. **`frontend/src/router/index.js`** - Route definitions
9. **`init-db.sql`** - Complete database schema
10. **`db.sh`** - Database management helper script

## Database Helper Script

The `./db.sh` script provides easy database management:

```bash
./db.sh start    # Start PostgreSQL container
./db.sh stop     # Stop container
./db.sh status   # Check if running
./db.sh psql     # Connect with psql
./db.sh logs     # View logs
./db.sh backup   # Create backup
./db.sh restore  # Restore from backup
./db.sh reset    # Fresh start (DELETES DATA)
```

## Common Patterns

### Backend Route Pattern
```javascript
router.get('/endpoint', async (req, res) => {
  try {
    const { rows } = await pool.query('SELECT * FROM books');
    res.json(rows);
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: 'Error message' });
  }
});
```

### Frontend API Call Pattern
```javascript
import { ref, onMounted } from 'vue'
import { bookApi } from '../services/api'

const data = ref([])
const loading = ref(true)
const error = ref(null)

const fetchData = async () => {
  loading.value = true
  try {
    const response = await bookApi.getAll()
    data.value = response.data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
```

### PostgreSQL Array Queries
```sql
-- Insert array
INSERT INTO books (genres) VALUES (ARRAY['Fiction', 'Fantasy']);

-- Query array elements
SELECT * FROM books WHERE 'Fiction' = ANY(genres);

-- Expand array to rows
SELECT UNNEST(genres) as genre FROM books;

-- Count by genre
SELECT genre, COUNT(*) 
FROM books, UNNEST(genres) as genre 
GROUP BY genre;
```

## Debugging Tips

1. **Backend errors:** Check terminal running `npm run dev`
2. **Frontend errors:** Check browser DevTools console (F12)
3. **Database issues:** Run `./db.sh logs` or `./db.sh psql`
4. **CORS errors:** Verify `CORS_ORIGIN` in backend `.env` matches frontend URL
5. **API issues:** Use `curl` or the `./test-api.sh` script
6. **Env var changes:** Must restart dev servers (Ctrl+C, then `npm run dev`)

## Testing

### Manual Testing Flow
1. Start database: `./db.sh start`
2. Start backend: `cd backend && npm run dev`
3. Start frontend: `cd frontend && npm run dev`
4. Open http://localhost:5173
5. Test: Add book → View in library → Edit → Delete

### API Testing
```bash
# Health check
curl http://localhost:3000/api/health

# Search books
curl "http://localhost:3000/api/books/search?q=tolkien"

# Get all books
curl http://localhost:3000/api/books

# Run test script
./test-api.sh
```

## Known Gotchas

1. **Vite env vars:** Changes require restart, not hot reload
2. **Tailwind v4:** Don't use old v3 config files or PostCSS setup
3. **PostgreSQL arrays:** Use `ARRAY[]` syntax, not `{}`
4. **Date inputs:** Browser locale affects display, but value is always ISO
5. **CORS:** Must match exact origin including protocol and port
6. **Podman vs Docker:** Commands are identical, just use `podman` instead of `docker`

## Recent Changes

- Migrated to Tailwind CSS v4 with Vite plugin
- Converted all components to Composition API (`<script setup>`)
- Added reading date tracking (start_date, end_date)
- Added dashboard statistics with monthly chart
- Implemented book editing functionality
- Added ISBN detection utility
- Integrated Google Books API search

## Future Enhancements (TODO)

- Re-read tracking (multiple reading sessions per book)
- Reading goals and progress tracking
- Book ratings and reviews
- Reading streaks
- Export data (CSV/JSON)
- Book recommendations
- Social features (share reading lists)

## Git Workflow

- Single monorepo (not separate repos)
- Main branch: `main`
- Feature branches for new features
- Commit messages should mention affected area (Backend:, Frontend:, etc.)

## Deployment Notes

- Monorepo structure optimized for Coolify
- Backend builds from `/backend` directory
- Frontend builds from `/frontend` directory
- Separate environment variables for each service
- Database can be external or Coolify-managed PostgreSQL

---

## Quick Reference Commands

```bash
# Start everything
./db.sh start
cd backend && npm run dev     # Terminal 1
cd frontend && npm run dev    # Terminal 2

# Check status
./db.sh status
curl http://localhost:3000/api/health
curl http://localhost:3000/api/db-health

# Database access
./db.sh psql
# Inside psql: \dt (list tables), \d books (describe table), \q (quit)

# Reset database (CAUTION: deletes all data)
./db.sh reset
```

---

**For AI Assistants:**
- Always respect the established patterns (Composition API, Tailwind v4, PostgreSQL arrays)
- Check existing code before suggesting alternatives
- Test changes with `curl` for backend, browser for frontend
- Update relevant documentation when adding features
- Follow Vue 3 and modern JavaScript best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BSBjorn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BSBjorn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
