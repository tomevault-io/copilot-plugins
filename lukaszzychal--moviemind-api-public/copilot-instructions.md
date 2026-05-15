## moviemind-api-public

> > **This file contains context about the MovieMind API project for the AI assistant.**

# MovieMind API - Project Context

> **This file contains context about the MovieMind API project for the AI assistant.**
> 
> It is automatically loaded by Cursor IDE when the "Include CLAUDE.md in context" option is enabled in settings.

---

## 🎯 Project Overview

MovieMind API is a RESTful API for generating and storing unique descriptions of movies, series, and actors using AI technology. The project creates original, AI-generated content instead of copying content from IMDb or TMDb.

---

## 🏗️ Technology Stack

### Backend
- **Framework:** Laravel 12
- **PHP:** 8.2+
- **Database:** PostgreSQL (production and tests)
- **Cache:** Redis
- **Queue:** Laravel Horizon (asynchronous processing)
- **AI Integration:** OpenAI API (gpt-4o-mini)

### Frontend
- **Stack:** Vue 3, Vite, Tailwind CSS (SPA in `frontend/`)
- **CI:** `.github/workflows/frontend.yml` (lint + build on `frontend/**`)

### Development Tools
- **Tests:** PHPUnit (Feature Tests + Unit Tests)
- **Formatting:** Laravel Pint (PSR-12)
- **Static analysis:** PHPStan (level 5)
- **Security:** GitLeaks (secret detection)
- **Documentation:** OpenAPI/Swagger
- **Local Environment:** Docker Compose (mandatory - see `.cursor/rules/090-docker-development.mdc`)

---

## 🐳 Local Development

### Docker is Mandatory

**ALWAYS use Docker for local development. NEVER use `php artisan serve` directly on host.**

**Quick Start:**
```bash
# Start all services
docker compose up -d

# Setup application (first time)
docker compose exec php composer install
docker compose exec php php artisan key:generate
docker compose exec php php artisan migrate --seed

# Access application
# API: http://localhost:8000
```

**Run commands inside PHP container:**
```bash
docker compose exec php php artisan migrate
docker compose exec php php artisan test
docker compose exec php vendor/bin/pint
```

**Run tests (mandatory: inside Docker):** Testy używają PostgreSQL (`DB_HOST=db`). Na hoście „db” nie istnieje, więc **nie uruchamiaj** `cd api && composer test` na hoście – dostaniesz błędy połączenia z bazą. Uruchamiaj testy **w kontenerze**:
```bash
# Z roota repozytorium (zalecane):
docker compose exec php composer test

# Lub bezpośrednio:
docker compose exec php bash run-tests.sh
```
Testy uruchamiane są **równolegle** (`--parallel`) – Laravel tworzy osobne bazy testowe (np. `moviemind_test_1`, `moviemind_test_2`) dla każdego procesu. Użytkownik bazy (np. `moviemind` w Dockerze) musi mieć prawo tworzenia baz (CREATEDB).
Z katalogu `api/` możesz też wejść do kontenera i tam uruchomić `composer test`. Pierwsze uruchomienie może trwać dłużej (migracje). Jeśli `docker compose exec php php artisan test` długo nic nie wyświetla – poczekaj ok. 30–60 s (RefreshDatabase + migracje), albo uruchom z `--verbose`.

**Why Docker is required:**
- Matches production environment (PostgreSQL, Redis, PHP-FPM, Nginx)
- Prevents "works on my machine" issues
- Ensures consistency between local, CI, and production
- See `.cursor/rules/090-docker-development.mdc` for full details

---

## 📁 Project Structure

### Main Structure
```
api/                          # Laravel application (backend)
├── app/
│   ├── Enums/               # Enumerations (Language, EntityType, etc.)
│   ├── Events/              # Laravel Events
│   ├── Features/            # Feature-based code
│   ├── Helpers/             # Helper functions
│   ├── Http/
│   │   ├── Controllers/     # API Controllers
│   │   ├── Requests/        # Request validators
│   │   └── Resources/        # API Resources
│   ├── Jobs/                # Queue Jobs (ShouldQueue)
│   ├── Listeners/           # Event Listeners
│   ├── Models/              # Eloquent Models
│   ├── Repositories/        # Repository pattern
│   └── Services/            # Business logic services
├── config/                  # Laravel configuration
├── database/
│   ├── migrations/          # Database migrations
│   └── seeders/             # Seeders
├── routes/
│   └── api.php              # Route definitions
└── tests/
    ├── Feature/             # Feature tests (API endpoints)
    └── Unit/                # Unit tests (classes, services)

frontend/                    # Vue 3 + Vite + Tailwind (SPA for API)
├── src/
│   ├── App.vue
│   ├── main.js
│   ├── style.css            # Tailwind directives
│   └── config.js            # VITE_API_BASE_URL etc.
├── public/
├── index.html
├── package.json
├── vite.config.js           # Dev proxy /api → backend
├── tailwind.config.js
└── README.md                # Setup, dev, build
```

The frontend is a separate app that consumes the MovieMind API. Run it with `cd frontend && npm run dev` (see `frontend/README.md`). CI: `.github/workflows/frontend.yml` runs lint and build on changes under `frontend/`.

---

## 🗄️ Data Model

### Main Tables

**Movies**
- `id` (PK)
- `title`
- `release_year`
- `director`
- `genres` (array)
- `default_description_id` (FK)

**Movie Descriptions**
- `id` (PK)
- `movie_id` (FK)
- `locale` (pl-PL, en-US, etc.)
- `text` (AI-generated content)
- `context_tag` (modern, critical, humorous)
- `origin` (GENERATED/TRANSLATED)
- `ai_model` (gpt-4o-mini)
- `created_at`

**Actors & Bios**
- Similar structure to Movies/Descriptions
- `actors` - basic actor data
- `actor_bios` - AI-generated biographies

**Jobs (Async Processing)**
- `id` (PK)
- `entity_type` (MOVIE, ACTOR)
- `entity_id`
- `locale`
- `status` (PENDING, DONE, FAILED)
- `payload_json`
- `created_at`

---

## 🔄 Architecture and Flow

### Current Flow (Laravel Events + Jobs)
```
Controller
  ↓
Event (e.g. MovieGenerationRequested)
  ↓
Listener (QueueMovieGenerationJob)
  ↓
Job (GenerateMovieJob implements ShouldQueue)
  ↓
Queue Worker (Laravel Horizon)
  ↓
AI Service (OpenAI API)
  ↓
Database (save result)
```

### Design Patterns
- **Repository Pattern** - data access abstraction
- **Service Layer** - business logic
- **Event-Driven** - Events + Listeners for asynchronous operations
- **Queue Jobs** - long-running operations (AI generation)

---

## 🧪 Tests

### Test Types

1. **Feature Tests** (`tests/Feature/`)
   - Test API endpoints
   - Use test database (PostgreSQL via Docker; see docs/knowledge/reference/TESTING_DATABASE.md)
   - Example: `MovieControllerTest`, `GenerateApiTest`

2. **Unit Tests** (`tests/Unit/`)
   - Test individual classes and methods
   - Fast, isolated
   - Example: `MovieServiceTest`, `ValidationHelperTest`

### TDD Workflow
- **RED** - Write a test that defines the requirement
- **GREEN** - Write minimal code to pass the test
- **REFACTOR** - Improve code while keeping tests passing

**IMPORTANT:** Always write tests before implementation!

---

## 📝 Naming Conventions

### Classes
- **Controllers:** `MovieController`, `PersonController` (suffix: Controller)
- **Models:** `Movie`, `MovieDescription`, `Actor` (PascalCase, singular)
- **Services:** `MovieService`, `AiService` (suffix: Service)
- **Jobs:** `GenerateMovieJob`, `GenerateActorBioJob` (suffix: Job)
- **Events:** `MovieGenerationRequested` (verb in past tense)
- **Listeners:** `QueueMovieGenerationJob` (action + object)
- **Requests:** `StoreMovieRequest`, `UpdateMovieRequest` (action + object + Request)
- **Resources:** `MovieResource`, `ActorResource` (object + Resource)

### Methods
- **Controllers:** `index()`, `show()`, `store()`, `update()`, `destroy()` (standard REST)
- **Services:** `create()`, `find()`, `update()`, `delete()`, `generate()` (business actions)
- **Tests:** `test_can_create_movie()` (snake_case, prefix: test_)

### Files
- **Migrations:** `2024_01_01_000000_create_movies_table.php` (timestamp_description)
- **Seeders:** `MovieSeeder`, `ActorSeeder` (object + Seeder)

---

## 🔧 Pre-Commit Workflow

Before each commit you MUST run:

1. **Laravel Pint** - formatting
   ```bash
   cd api && vendor/bin/pint
   ```

2. **PHPStan** - static analysis
   ```bash
   cd api && vendor/bin/phpstan analyse --memory-limit=2G
   ```

3. **PHPUnit** - tests
   ```bash
   cd api && php artisan test
   ```

4. **GitLeaks** - secret detection
   ```bash
   gitleaks protect --source . --verbose --no-banner
   ```

5. **Composer Audit** - security audit
   ```bash
   cd api && composer audit
   ```

---

## 🎯 Coding Principles

### SOLID (apply pragmatically)
- **SRP** - One class = one responsibility
- **DIP** - Depend on abstractions (interfaces)

### DRY
- Refactor duplication when it occurs in 3+ places
- Don't overdo abstraction

### Type Safety
- Always use `declare(strict_types=1);` in PHP files
- Always specify type hints for parameters and return types
- Use types instead of `mixed` where possible

### Laravel Conventions
- Use Eloquent Models instead of Query Builder when possible
- Use Form Requests for validation
- Use API Resources for responses
- Use Events + Jobs for asynchronous operations

### Controller Architecture (MANDATORY)
- **Thin Controllers** - Controllers MUST be thin (max 20-30 lines per method)
- **Delegate to Actions** - Use Actions (`App\Actions\*`) for business logic
- **Delegate to Services** - Use Services (`App\Services\*`) for external dependencies, caching, complex operations
- **Use Repositories** - Use Repositories (`App\Repositories\*`) for database access
- **Use Response Formatters** - Use Response Formatters (`App\Http\Responses\*`) for consistent API responses
- **NEVER** put business logic, database queries, external API calls, or complex transformations directly in controllers
- See `docs/cursor-rules/pl/controller-architecture.mdc` for detailed patterns and examples

---

## 📚 Key Documentation Files

- **AI Rules:** `.cursor/rules/*.mdc` (rules in new format) + `docs/AI_AGENT_CONTEXT_RULES.md` (details)
- **Tasks:** `docs/issue/TASKS.md` - ⭐ START HERE
- **Tests:** `docs/TESTING_STRATEGY.md`
- **Tools:** `docs/CODE_QUALITY_TOOLS.md`
- **Architecture:** `docs/ARCHITECTURE_ANALYSIS.md`
- **Cursor explanation:** `docs/CURSOR_RULES_EXPLANATION.md`
- **Technical (rate limits, resilience, observability):** `docs/knowledge/technical/RATE_LIMITING_STRATEGY.md`, `docs/knowledge/technical/CIRCUIT_BREAKER_ANALYSIS.md`, `docs/knowledge/technical/LOGGING_AND_OBSERVABILITY.md`

---

## 🚀 API Endpoints

### Main Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/movies` | List of movies (with pagination, filtering) |
| `GET` | `/api/v1/movies/{id}` | Movie details + AI description |
| `POST` | `/api/v1/generate` | Trigger AI generation |
| `GET` | `/api/v1/jobs/{id}` | Generation job status |

### Examples

```bash
# Get movie
GET /api/v1/movies/123

# Trigger generation
POST /api/v1/generate
{
  "entity_type": "MOVIE",
  "entity_id": 123,
  "locale": "pl-PL",
  "context_tag": "modern"
}
```

---

## 🔐 Security

### Before Commit
- ✅ Check GitLeaks (zero secrets)
- ✅ Check Composer Audit (critical vulnerabilities)
- ✅ Use environment variables for API keys
- ✅ Never commit `.env` with real values

### Secrets
- OpenAI API keys: `OPENAI_API_KEY` (environment variable)
- Database passwords: in `.env` (not in repo)
- All secrets: in `.env` or environment variables

---

## 💡 Important Notes

1. **Docker is Mandatory** - ALWAYS use Docker for local development. NEVER use `php artisan serve` directly on host.
   - Local environment must match production (PostgreSQL, Redis via Docker)
   - Prevents test failures in production due to environment differences
   - See `.cursor/rules/090-docker-development.mdc` for full requirements
2. **TDD** - Test before code, ALWAYS. NEVER write implementation code before tests. This is MANDATORY with NO EXCEPTIONS.
   - Write test first (RED) → Run test (should FAIL) → Write code (GREEN) → Run test (should PASS) → Refactor
   - See `docs/cursor-rules/pl/testing.mdc` for detailed TDD workflow
3. **Thin Controllers** - Controllers MUST be thin (max 20-30 lines per method)
   - Delegate business logic to Actions (`App\Actions\*`) or Services (`App\Services\*`)
   - Controllers should ONLY: validate requests, delegate to Actions/Services, format responses
   - NEVER put business logic, database queries, external API calls, or complex transformations in controllers
   - See `docs/cursor-rules/pl/controller-architecture.mdc` for detailed patterns
4. **Tools** - Pint, PHPStan, tests before commit
5. **Readability** - Code must be understandable to others
6. **Pragmatism** - Principles are tools, not goals in themselves
7. **Tasks** - Always start from `docs/issue/TASKS.md`

---

**This file is updated as the project evolves. Check `docs/` for detailed information.**

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
