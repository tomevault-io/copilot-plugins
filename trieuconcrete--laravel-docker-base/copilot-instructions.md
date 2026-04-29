## laravel-docker-base

> Laravel Docker Base - Project conventions, tech stack và coding standards


# Laravel Docker Base — Project Rules

## Tech Stack
- **Framework**: Laravel 12.x
- **PHP**: 8.3 (chạy trong Docker container `laravel_app`)
- **Web Server**: Nginx Alpine — port ngoài `8000`, port trong `80`
- **Database**: MySQL 8.0 — port ngoài `3307`, port trong `3306`
- **Cache / Session / Queue**: Redis Alpine — port `6379`
- **Email testing**: Mailhog — Web UI `http://localhost:8025`, SMTP `1025`
- **Container network**: Bridge network tên `laravel`

## Project Structure
```
laravel-docker-base/
├── src/                          ← Laravel source code (root của app)
│   ├── app/
│   │   ├── Console/Commands/     ← Artisan commands (Database, Project, User)
│   │   ├── Http/Controllers/
│   │   │   └── Admin/            ← Admin controllers
│   │   ├── Models/
│   │   └── Providers/
│   ├── config/                   ← Laravel config files
│   ├── database/
│   │   ├── migrations/
│   │   ├── seeders/
│   │   └── factories/
│   ├── resources/views/
│   │   ├── layouts/              ← admin.blade.php, admin-professional.blade.php
│   │   ├── auth/                 ← login, login-simple, login-professional
│   │   ├── partials/
│   │   └── admin/
│   │       └── modules/          ← cms, crm, ecommerce, jobs, users
│   └── routes/web.php
├── docker/
│   ├── nginx/default.conf
│   └── php/Dockerfile + php.ini
├── docker-compose.yml            ← Development
├── docker-compose.prod.yml       ← Production overrides
└── Makefile                      ← 20+ utility commands
```

## Database Connection Rules
- **Trong container / Laravel .env**: `DB_HOST=mysql`, `DB_PORT=3306`
- **Từ host machine (GUI tools)**: `localhost:3307`
- KHÔNG dùng `localhost` hoặc `127.0.0.1` làm `DB_HOST` trong `.env`
- Tương tự Redis: `REDIS_HOST=redis`, Mail: `MAIL_HOST=mailhog`

## Route Structure
- Admin routes prefix: `/admin`, name prefix: `admin.`
- Module groups: `admin.cms.*`, `admin.crm.*`, `admin.products.*`, `admin.orders.*`, `admin.customers.*`, `admin.jobs.*`, `admin.users.*`
- Auth routes: `/login`, `/register`, `/logout`, `/forgot-password`

## Admin Blade Layout
- Admin pages: extend `layouts.admin` hoặc `layouts.admin-professional`
- Sidebar: partial `partials.sidebar-menu`
- Module views: `admin.modules.{module}.{action}`

## Makefile Commands (LUÔN dùng thay vì lệnh thủ công)
```bash
make up              # docker-compose up -d
make down            # docker-compose down
make shell           # Vào container PHP
make artisan cmd=""  # Chạy artisan command
make composer cmd="" # Chạy composer command
make migrate         # php artisan migrate
make seed            # php artisan db:seed
make cache-clear     # php artisan cache:clear
make optimize        # php artisan optimize
make perm            # Fix storage permissions
make logs            # Xem logs tất cả containers
make logs-app        # Xem logs PHP container
make backup          # Backup database
```

## Coding Conventions

### PHP / Laravel
- Dùng PHP 8.3+ features: readonly, named arguments, match expression, enums
- Tuân theo PSR-12
- Form validation: dùng Form Request classes, không validate trực tiếp trong controller
- Eloquent: dùng relationships, scopes — tránh raw SQL trừ khi cần thiết
- Khi tạo resource: `make artisan cmd="make:model Product -mcr"` (model + migration + controller + resource)

### Docker & Artisan
- KHÔNG chạy `php artisan` trực tiếp trên host → chạy trong container qua `make artisan`
- KHÔNG sửa files trong `src/bootstrap/cache/` — auto-generated, sẽ bị overwrite
- KHÔNG sửa files trong `src/storage/` — runtime files

### Khi thêm package mới
```bash
make composer cmd="require vendor/package"
make artisan cmd="vendor:publish --provider=VendorServiceProvider"
make cache-clear
```

## Response Guidelines
- Khi tạo Controller mới: đặt đúng namespace, dùng Form Request cho validation
- Khi tạo Migration: gợi ý chạy `make artisan cmd="make:migration create_table_name"`
- Khi debug Docker: kiểm tra `make logs-app` trước tiên
- Luôn gợi ý Makefile commands thay vì docker-compose commands dài
- Khi hỏi về kết nối database: phân biệt rõ kết nối từ trong container vs từ host

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trieuconcrete) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
