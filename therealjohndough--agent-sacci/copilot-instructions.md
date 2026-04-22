## agent-sacci

> This file provides essential context for AI assistants working on this repository.

# CLAUDE.md — Sacci Brand Hub

This file provides essential context for AI assistants working on this repository.

## Project Overview

**Sacci Brand Hub** is a lightweight PHP 8.2 CMS (Content Management System) built for House of Sacci, a cannabis retailer. It provides:

- Internal brand portal for staff to manage tickets, assets, and content blocks
- Retail partner portal for organization-specific asset access
- Role-based access control (RBAC) for multi-tenant access management
- Digital asset management with controlled file downloads

**Deployment target:** SiteGround shared hosting (no Docker, no Node.js build step, no heavy framework).

---

## Repository Structure

```
agent-sacci/
├── sacci_brand_hub/          # Main application root (document root on server)
│   ├── index.php             # Front controller — registers all routes and boots app
│   ├── composer.json         # Dependency manifest (PHPMailer only)
│   ├── .htaccess             # Apache: routes all requests through index.php
│   ├── config/
│   │   └── config.php        # loadEnv() and env() helpers — reads .env file
│   ├── core/                 # Framework-like base classes
│   │   ├── Auth.php          # Static session auth + RBAC permission checks
│   │   ├── Csrf.php          # Per-session CSRF token generation & validation
│   │   ├── Database.php      # Singleton PDO wrapper (MySQL 8, utf8mb4)
│   │   ├── Router.php        # Simple method+path → controller dispatch
│   │   └── View.php          # Output-buffered PHP template renderer with layout
│   ├── app/
│   │   ├── controllers/      # One controller per endpoint group
│   │   │   ├── BaseController.php      # requireLogin(), render(), csrfToken()
│   │   │   ├── AuthController.php      # GET+POST /login, GET /logout
│   │   │   ├── DashboardController.php # GET /app
│   │   │   ├── TicketController.php    # GET /tickets
│   │   │   ├── AssetController.php     # GET /assets, GET /assets/download
│   │   │   ├── ContentController.php   # GET /content (admin-only)
│   │   │   └── PortalController.php    # GET /portal (retailer-only)
│   │   ├── models/           # Active-record style models extending BaseModel
│   │   │   ├── BaseModel.php           # find(), findBy(), create(), update()
│   │   │   ├── User.php
│   │   │   ├── Role.php
│   │   │   ├── Permission.php
│   │   │   ├── Ticket.php
│   │   │   ├── TicketComment.php
│   │   │   ├── TicketActivity.php
│   │   │   ├── Asset.php
│   │   │   ├── ContentBlock.php
│   │   │   ├── Organization.php
│   │   │   ├── Setting.php
│   │   │   └── AuditLog.php
│   │   └── views/            # PHP templates (no Twig/Blade — plain PHP)
│   │       ├── layout/main.php         # Master layout (navbar, CSS vars, Montserrat)
│   │       ├── auth/login.php
│   │       ├── app/dashboard.php
│   │       ├── app/tickets/index.php
│   │       ├── app/tickets/show.php
│   │       ├── app/assets/index.php
│   │       ├── app/assets/show.php
│   │       ├── app/content/index.php
│   │       └── portal/dashboard.php
│   ├── migrations/
│   │   ├── 001_create_tables.php  # Full schema creation (13 tables)
│   │   └── 002_seed_data.php      # Roles, permissions, sample data
│   ├── install/
│   │   └── index.php         # Web installer wizard (deletes itself after success)
│   └── storage/              # File uploads — outside web root, .htaccess denies direct access
├── README.md                 # Project documentation and deployment guide
└── LICENSE                   # MIT
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Language | PHP 8.2+ |
| Database | MySQL 8 (PDO, utf8mb4) |
| Template engine | Plain PHP (output buffering) |
| External dependency | PHPMailer ^6.9 |
| Web server | Apache (via `.htaccess`) |
| Autoloading | PSR-4 via custom `spl_autoload_register` (no Composer autoloader at runtime) |
| Frontend | Plain HTML + CSS custom properties, Montserrat from Google Fonts, no JS framework |

There is **no build step**, **no Node.js**, **no TypeScript**, and **no test suite**.

---

## Environment Configuration

The app reads from a `.env` file in `sacci_brand_hub/`. Environment variables are loaded by `Config\loadEnv()` and accessed via `Config\env($key, $default)`.

Required environment variables:

```ini
APP_NAME=Sacci Brand Hub
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=sacci_brand_hub
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
```

Optional (SMTP for PHPMailer):

```ini
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=user@example.com
MAIL_PASSWORD=secret
MAIL_FROM=noreply@houseofsacci.com
```

During development, copy `.env.example` (if present) or create `.env` manually. The web installer (`install/index.php`) can write this file via the browser.

---

## Local Development Setup

No package manager scripts exist. Setup is manual:

```bash
# 1. Install PHP dependencies
cd sacci_brand_hub
composer install

# 2. Create .env file with DB credentials (see above)
cp .env.example .env   # then edit values

# 3. Run migrations (requires DB to already exist)
php migrations/001_create_tables.php
php migrations/002_seed_data.php

# 4. Start a dev server (PHP built-in server)
php -S localhost:8000 -t sacci_brand_hub sacci_brand_hub/index.php
```

Alternatively, use the web installer at `/install/` after pointing a web server at the document root.

---

## Routing

All HTTP requests flow through `sacci_brand_hub/index.php`. Routes are registered manually using `Router::add(method, path, handler)`:

| Method | Path | Controller@action | Notes |
|---|---|---|---|
| GET | `/login` | `AuthController@login` | Show login form |
| POST | `/login` | `AuthController@login` | Process credentials |
| GET | `/logout` | `AuthController@logout` | Destroy session |
| GET | `/app` | `DashboardController@index` | Internal user dashboard |
| GET | `/tickets` | `TicketController@index` | Ticket list |
| GET | `/assets` | `AssetController@index` | Asset library |
| GET | `/assets/download` | `AssetController@download` | Controlled file download |
| GET | `/portal` | `PortalController@index` | Retailer partner portal |
| GET | `/content` | `ContentController@index` | CMS content blocks (admin) |

To add a new route, register it in `index.php` before `$router->dispatch()`.

**No dynamic URI segments** (e.g., `/tickets/{id}`) — the Router only matches exact paths. Record IDs are passed as query parameters (e.g., `?id=5`).

---

## MVC Conventions

### Controllers

- Extend `App\Controllers\BaseController`
- Call `$this->requireLogin()` at the top of any protected action — redirects to `/login` if unauthenticated
- Render views via `$this->render('view/path', $data)` where `view/path` is relative to `app/views/` without `.php`
- Access CSRF token via `$this->csrfToken()`
- Check permissions via `Core\Auth::hasPermission('permission.name')`

```php
class TicketController extends BaseController
{
    public function index(): void
    {
        $this->requireLogin();
        if (!Auth::hasPermission('tickets.manage')) {
            http_response_code(403);
            echo 'Forbidden';
            return;
        }
        $tickets = Ticket::findBy(['assigned_to' => Auth::user()['id']]);
        $this->render('app/tickets/index', ['tickets' => $tickets]);
    }
}
```

### Models

- Extend `App\Models\BaseModel`
- Declare `protected static string $table = 'table_name'`
- Use `static::$table` (late static binding) for table name resolution
- All DB access goes through `Core\Database::getConnection()` (singleton PDO)
- Methods return associative arrays (`PDO::FETCH_ASSOC`), never objects

Inherited methods on `BaseModel`:
- `find(int $id): ?array` — fetch by primary key
- `findBy(array $conditions): array` — fetch all rows matching conditions
- `create(array $data): int` — insert and return new ID
- `update(int $id, array $data): void` — update columns by primary key

Models add their own static methods for queries not covered by the base (e.g., `User::findByEmail()`).

### Views

- Plain PHP templates stored in `app/views/`
- Rendered by `Core\View::render($view, $data)` which:
  1. Calls `extract($data)` — all `$data` keys become local variables
  2. Buffers the view template output
  3. Wraps it with `app/views/layout/main.php` — the layout receives the buffered output as `$content`
- Always escape output with `htmlspecialchars($value, ENT_QUOTES, 'UTF-8')`
- CSRF token in forms: `<input type="hidden" name="_csrf" value="<?= $csrf ?>">`

---

## Database Schema

13 tables, all `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4`:

| Table | Purpose |
|---|---|
| `users` | Accounts: `email`, `password_hash`, `organization_id` |
| `roles` | Named roles: `super_admin`, `admin`, `staff`, `retailer_manager`, `retailer_user` |
| `permissions` | Named permissions: `user.manage`, `content.manage`, `tickets.manage`, `assets.manage`, `organizations.manage` |
| `user_roles` | Many-to-many pivot: `user_id ↔ role_id` |
| `role_permissions` | Many-to-many pivot: `role_id ↔ permission_id` |
| `organizations` | Retail partner orgs: `name`, `contact_name`, `contact_email` |
| `tickets` | Requests: `title`, `description`, `request_type` (enum), `priority` (enum), `status` (enum), `assigned_to`, `requester_id` |
| `ticket_comments` | Discussion on tickets |
| `ticket_activities` | Activity log per ticket |
| `assets` | Brand files: `filepath`, `visibility` (public/internal/org), `org_id`, `category`, `brand` |
| `content_blocks` | CMS sections identified by `slug` |
| `settings` | Key-value configuration store |
| `audit_logs` | User action audit trail |

Migrations are plain PHP scripts run directly via CLI or the web installer. There is no migration runner or rollback mechanism — migrations are additive only.

---

## Authentication & Authorization

### Authentication

- Session-based: `$_SESSION['user_id']` is set on login
- `Core\Auth::check()` — returns true if session has a user ID
- `Core\Auth::user()` — returns current user as array (or null)
- `Core\Auth::login(int $userId)` — sets session user ID
- `Core\Auth::logout()` — unsets user ID and regenerates session ID

### RBAC Permissions

Permissions are checked dynamically by querying the DB:

```
Auth::hasPermission('assets.manage')
  → lookup user roles → lookup role permissions → check name match
```

Available permissions and their intended grants:

| Permission | super_admin | admin | staff | retailer_manager | retailer_user |
|---|---|---|---|---|---|
| `user.manage` | ✓ | | | | |
| `content.manage` | ✓ | ✓ | | | |
| `tickets.manage` | ✓ | ✓ | ✓ | | |
| `assets.manage` | ✓ | ✓ | ✓ | | |
| `organizations.manage` | ✓ | ✓ | | | |

### CSRF Protection

- `Core\Csrf::token()` — generates a random 64-char hex token stored in session
- `Core\Csrf::validate(string $token)` — uses `hash_equals()` for timing-safe comparison
- All POST forms must include `<input type="hidden" name="_csrf" value="...">` and validate on submission

---

## Security Conventions

These must be followed for all new code:

1. **SQL**: Always use PDO prepared statements. Never interpolate user input into SQL strings.
2. **Output**: Always escape with `htmlspecialchars()` before rendering user-controlled data in HTML.
3. **File downloads**: Serve files through `AssetController@download`, never expose `storage/` paths directly. Validate permissions before serving.
4. **File uploads**: Validate MIME type and file extension. Store in `storage/` (outside web root), never in a publicly accessible directory.
5. **CSRF**: Validate `$_POST['_csrf']` on every state-changing POST request.
6. **Authentication**: Call `$this->requireLogin()` at the start of every protected controller action.
7. **Password hashing**: Use `password_hash($password, PASSWORD_DEFAULT)` for hashing and `password_verify()` for checking — never store plaintext.

---

## UI / Frontend Conventions

The UI uses CSS custom properties defined in `app/views/layout/main.php`:

```css
--color-primary: #31935f;    /* Sacci green */
--color-accent:  #d4a837;    /* Sacci gold */
--color-bg:      #0d0d0d;    /* Very dark background */
--color-surface: #f5f0e8;    /* Light text / surface */
--color-card:    #1a1a1a;    /* Dark card background */
--color-muted:   #888880;    /* Muted gray */
```

- Font: Montserrat (Google Fonts, loaded in layout)
- No JavaScript framework — use progressive enhancement only
- Cards: dark charcoal background, gold border on hover
- Buttons: gold background (`--color-accent`) with dark text
- All forms include a hidden CSRF field and use dark inputs with gold submit buttons
- Navigation bar is sticky at top; active link highlighted in gold

---

## Content Blocks (CMS)

The `content_blocks` table stores editable CMS sections. Each block is identified by a `slug`:

| Slug | Content |
|---|---|
| `design-tokens` | Colors, typography, spacing tokens |
| `brand-system` | Brand identity, voice, typography rules |
| `asset-structure` | Folder hierarchy, naming conventions |
| `naming-rules` | File naming formula and examples |
| `sell-sheets` | Budtender sell sheets |
| `agent-instructions` | AI system prompt and operational rules |
| `retail-partner-templates` | Welcome brief, training checklist, templates |

---

## Known TODOs / Open Issues

- **`PortalController::index()`**: Assets are not yet filtered by organization or visibility. Retailer users currently see all assets — this must be fixed before production.
- **Brute-force protection**: No rate limiting exists on `POST /login`. Rate limiting or account lockout should be added.
- **Admin UI for role assignment**: Currently roles are assigned via seed data or direct DB edits. An admin UI is needed.
- **POST routes for mutations**: Most data-changing actions (creating tickets, uploading assets, editing content) are not yet implemented as POST routes — only read views exist. These need to be added in `index.php` and the respective controllers.
- **SMTP config**: PHPMailer is declared as a dependency but no mail-sending code is implemented yet.

---

## Adding New Features

### Adding a new route

1. Register in `sacci_brand_hub/index.php`:
   ```php
   $router->add('GET', '/new-path', [App\Controllers\NewController::class, 'action']);
   ```
2. Create the controller in `app/controllers/NewController.php` extending `BaseController`
3. Create view(s) in `app/views/new-path/`

### Adding a new model

1. Create `app/models/NewThing.php` extending `BaseModel`
2. Set `protected static string $table = 'new_things'`
3. Add any custom query methods as static methods returning arrays

### Adding a new database table

1. Add a new migration file `migrations/003_add_new_thing.php` (sequential numbering)
2. Use `Database::getConnection()` and `$pdo->exec()` for DDL
3. Document the table in this file under the Database Schema section
4. Run the migration manually: `php migrations/003_add_new_thing.php`

---

## Deployment

Target: SiteGround shared hosting with PHP 8.2 and MySQL 8.

1. Upload `sacci_brand_hub/` contents to the document root (e.g., `public_html/brandhub/`)
2. Place `storage/` **outside** the public root if the hosting plan allows
3. Run `composer install` (SSH or upload pre-built `vendor/`)
4. Visit `/install/` in the browser to write `.env`, run migrations, and create the super admin
5. Delete `install/` after successful setup
6. Ensure `storage/` is writable by the web server (`chmod 755`)

The `.htaccess` at the project root handles URL rewriting (`RewriteRule . index.php [L]`). Ensure `mod_rewrite` is enabled and `AllowOverride All` is set in Apache config.

---

## Git Workflow

- Main branch: `master`
- Feature/AI branches follow the pattern: `claude/<description>-<session-id>`
- No CI/CD pipeline exists — changes are deployed manually via FTP/SSH

Commit messages should be descriptive and in the imperative mood (e.g., "Add asset visibility filter to PortalController").

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/therealjohndough) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
