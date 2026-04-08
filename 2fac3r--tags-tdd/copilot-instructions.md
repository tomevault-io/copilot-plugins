## tags-tdd

> `tags-tdd` is a demonstration project for managing tags, built with a strong emphasis on **Test-Driven Development (TDD)** methodology. It illustrates how to implement basic CRUD operations (specifically Create and Delete) and model-level logic (like slug generation) using a test-first approach in the Laravel ecosystem.

# GEMINI.md - tags-tdd

## Project Overview
`tags-tdd` is a demonstration project for managing tags, built with a strong emphasis on **Test-Driven Development (TDD)** methodology. It illustrates how to implement basic CRUD operations (specifically Create and Delete) and model-level logic (like slug generation) using a test-first approach in the Laravel ecosystem.

### Core Technologies
- **Framework:** Laravel 8.x
- **Language:** PHP 7.3+ / 8.0+
- **Database:** MySQL (configured via `.env`)
- **Frontend:** TailwindCSS (via Mix/Webpack)
- **Testing:** PHPUnit (integrated with Laravel's testing suite)

### Architecture
The project follows the standard Laravel MVC architecture:
- **Models:** `App\Models\Tag` handles tag data and slug generation logic.
- **Controllers:** `App\Http\Controllers\TagController` manages the storage and deletion of tags.
- **Routes:** Defined in `routes/web.php`.
- **Tests:** Structured into `Unit/Models` for model-specific logic and `Feature/Http/Controllers` for HTTP request/response validation.

## Docker Setup
The project is now fully dockerized using Docker Compose.

### Services
- **app**: PHP 8.0-FPM container (Port 9000 internally).
- **nginx**: Web server (Exposed at [http://localhost:8080](http://localhost:8080)).
- **db**: MySQL 8.0 database (Exposed at port 33061 for local tools).

### Docker Commands
- **Start containers:**
  ```bash
  docker compose up -d
  ```
- **Stop containers:**
  ```bash
  docker compose down
  ```
- **Run Artisan commands:**
  ```bash
  docker compose exec app php artisan <command>
  ```
- **Run tests:**
  ```bash
  docker compose exec app php artisan test
  ```
- **Run Composer:**
  ```bash
  docker compose exec app composer <command>
  ```

### Initial Setup (Already completed)
1. `docker compose up -d --build`
2. `docker compose exec app composer install`
3. `docker compose exec app php artisan key:generate`
4. `docker compose exec app php artisan migrate`
5. `docker compose exec app php artisan db:seed`

## Development Conventions

### TDD Workflow
- **Test-First:** Always write a failing test in `tests/Feature` or `tests/Unit` before implementing the corresponding feature or fix.
- **Feature Tests:** Use these to drive the development of controller actions and routing. (e.g., `tests/Feature/Http/Controllers/TagControllerTest.php`).
- **Unit Tests:** Use these for isolated logic, such as model accessors or helper methods. (e.g., `tests/Unit/Models/TagTest.php`).

### Coding Standards
- **Eloquent Models:** Use `protected $fillable` for mass assignment. Logic for presentation (like slugs) should be implemented as Eloquent Accessors (e.g., `getSlugAttribute`).
- **Validation:** Perform request validation within controller methods or via Form Requests.
- **Redirects:** Controller actions typically redirect back to the home page (`/`) after processing.

### Key Files
- `app/Models/Tag.php`: Contains the `Tag` model and slug generation.
- `app/Http/Controllers/TagController.php`: Handles `store` and `destroy` actions.
- `routes/web.php`: Defines the web endpoints.
- `database/migrations/2021_01_29_214804_create_tags_table.php`: Schema for the tags table.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2Fac3R)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2Fac3R)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
