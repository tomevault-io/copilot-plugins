## 090-docker-development

> **ALWAYS use Docker for local development. NEVER use `php artisan serve` or run PHP/Laravel directly on the host system.**

# Docker Development Environment

## Mandatory Docker Usage

**ALWAYS use Docker for local development. NEVER use `php artisan serve` or run PHP/Laravel directly on the host system.**

### Why Docker is Required

1. **Consistency with Production**
   - Production uses PostgreSQL, Redis, PHP-FPM, Nginx in containers
   - Local environment must match production to catch issues early
   - Prevents "works on my machine" problems

2. **Database Consistency**
   - Production: PostgreSQL
   - Tests: SQLite (for speed) + PostgreSQL (for production-like tests)
   - Local development: **MUST use PostgreSQL** (via Docker)
   - Using SQLite locally can hide PostgreSQL-specific issues

3. **Service Dependencies**
   - Application requires PostgreSQL and Redis
   - Docker Compose provides all services in one command
   - No need to install/manage PostgreSQL/Redis locally

### Commands to Use

**Start services:**
```bash
docker compose up -d
```

**Stop services:**
```bash
docker compose down
```

**Run commands inside PHP container:**
```bash
docker compose exec php <command>
# Examples:
docker compose exec php composer install
docker compose exec php php artisan migrate
docker compose exec php php artisan test
```

**View logs:**
```bash
docker compose logs -f php
docker compose logs -f horizon
docker compose logs -f nginx
```

### Commands to NEVER Use

❌ **DO NOT run these commands:**
- `php artisan serve` (on host)
- `composer run local:up` (blocked in composer.json)
- `composer run dev` (blocked in composer.json)
- Any command that runs PHP/Laravel directly on host

❌ **DO NOT configure `.env` for:**
- `DB_HOST=localhost` (use `db` - Docker service name)
- `REDIS_HOST=localhost` (use `redis` - Docker service name)
- Local PostgreSQL/Redis installations

### Docker Compose Services

The `compose.yml` provides:
- **php**: PHP-FPM container with Laravel application
- **nginx**: Web server (port 8000)
- **db**: PostgreSQL database
- **redis**: Redis cache/queue
- **horizon**: Laravel Horizon for queue processing

### Environment Variables

Use `env/local.env.example` as template for `api/.env`:
- `DB_HOST=db` (Docker service name, NOT localhost)
- `REDIS_HOST=redis` (Docker service name, NOT localhost)
- `APP_URL=http://localhost:8000`

### Testing

Tests can use SQLite (`:memory:`) for speed, but:
- **Feature tests** should also run with PostgreSQL in CI
- **Local development** must use PostgreSQL (via Docker)
- Never develop against SQLite if production uses PostgreSQL

### Enforcement

- Scripts `local:up` and `dev` in `composer.json` are blocked (throw exceptions)
- README.md clearly states Docker is required
- CI uses Docker services (PostgreSQL, Redis)

### When Docker is NOT Running

If Docker is not running and user needs to work:
1. **Start Docker Compose:**
   ```bash
   docker compose up -d
   ```

2. **Wait for services to be ready:**
   ```bash
   docker compose ps
   ```

3. **Run setup commands:**
   ```bash
   docker compose exec php composer install
   docker compose exec php php artisan migrate --seed
   ```

4. **Access application:**
   - API: `http://localhost:8000`
   - Horizon: `http://localhost:8000/horizon` (if configured)

### Troubleshooting

**If application doesn't start:**
- Check Docker is running: `docker ps`
- Check containers are up: `docker compose ps`
- Check logs: `docker compose logs`

**If port 8000 is already in use:**
- Check what's using it: `lsof -i :8000`
- Kill process or stop conflicting service
- Ensure no `php artisan serve` is running

**If database connection fails:**
- Verify `.env` has `DB_HOST=db` (not `localhost`)
- Check PostgreSQL container is running: `docker compose ps db`
- Check logs: `docker compose logs db`

---

**Remember: Docker is not optional - it's mandatory for local development.**

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
