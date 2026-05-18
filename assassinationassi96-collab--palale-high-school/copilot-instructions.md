## palale-high-school

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Laravel 12 school management system with role-based access control for managing admissions, enrollments, grades, schedules, and school operations. Built with PHP 8.2, Vite, and Tailwind CSS.

## Development Commands

### Initial Setup
```bash
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan db:seed
npm install
npm run build
```

Or use the composer script:
```bash
composer run setup
```

### Development Server
Start all services (web server, queue worker, logs, and Vite):
```bash
composer run dev
```

This runs concurrently:
- `php artisan serve` - Web server (http://localhost:8000)
- `php artisan queue:listen` - Queue worker
- `php artisan pail` - Real-time logs
- `npm run dev` - Vite dev server with HMR

### Individual Services
```bash
php artisan serve          # Start web server only
npm run dev                # Start Vite dev server
npm run build              # Build production assets
php artisan queue:listen   # Process queue jobs
php artisan pail           # View logs
```

### Database
```bash
php artisan migrate               # Run migrations
php artisan migrate:fresh --seed  # Fresh database with seeders
php artisan db:seed               # Run seeders
php artisan db:seed --class=UserSeeder  # Run specific seeder
```

Default database: SQLite (`database/database.sqlite`)

### Testing
```bash
composer run test          # Run full test suite
php artisan test           # Alternative test command
php artisan test --filter=ExampleTest  # Run specific test
```

### Code Quality
```bash
vendor/bin/pint            # Format code with Laravel Pint
php artisan config:clear   # Clear config cache
php artisan route:list     # View all routes
```

## Architecture

### Role-Based Access Control

Four user roles with distinct permissions:
- `superadmin` - Full system access, bypasses all role restrictions
- `registrar` - Student records, enrollment, admissions, class management
- `teacher` - Grades, schedules, advisory students, evaluations
- `student` - View grades, schedules, submit documents

Role authorization handled by `RoleMiddleware` (app/Http/Middleware/RoleMiddleware.php). Routes protected with `role:` middleware. User status must be "Active" to access authenticated routes.

### Controller Organization

Controllers are namespaced by role:
- `App\Http\Controllers\Auth\*` - Authentication (login, activation, password)
- `App\Http\Controllers\Registrar\*` - Enrollment, admissions, class/grade management
- `App\Http\Controllers\SuperAdmin\*` - User management, evaluations
- `App\Http\Controllers\Teacher\*` - Teaching, grades, schedules, resources
- `App\Http\Controllers\Student\*` - Student portal features

Public controllers (Home, Page, Faculty, News, Program) have no namespace prefix.

### Key Models and Relationships

**User Model** (app/Models/User.php):
- Central authentication model with role-based helper methods (`isAdmin()`, `isTeacher()`, etc.)
- Polymorphic relationships: `faculty()`, `student()`, `grades()`, `latestEnrollment()`
- Teachers link to `advisoryStudents()` and `evaluationsReceived()`
- Students link to `evaluationsSubmitted()`

**Student-Enrollment Flow**:
- User (role='student') → Student (LRN, personal info) → Enrollment (academic year/term) → SchoolClass → Grades
- Access via `User::student()` and `User::latestEnrollment()`

**Academic Structure**:
- SchoolClass - Sections with year_level, section, adviser_id
- Schedule - Class schedules with room assignments
- Grade - Student grades per subject
- EnrollmentSetting - School year and enrollment period configuration

### Frontend Stack

- **Views**: Blade templates in `resources/views/`
  - `layouts/app.blade.php` - Public site layout
  - `layouts/portal.blade.php` - Authenticated portal layout
  - `portal/{role}/*` - Role-specific dashboards and features
- **Assets**: Vite + Tailwind CSS v3
  - Entry: `resources/css/app.css` and `resources/js/app.js`
  - Build: `npm run build` outputs to `public/build/`

### Routing Conventions

Routes in `routes/web.php` organized by access level:
1. Public routes (home, about, programs, faculty, news)
2. Auth routes (login, activation, password reset)
3. Role-protected groups with `role:` middleware

Route naming: `{role}.{feature}.{action}` (e.g., `registrar.enrollment.store`, `student.grades.index`)

## Database Schema Notes

- Primary keys: `id` (auto-increment)
- Timestamps: `created_at`, `updated_at` on most tables
- Soft deletes where applicable
- Users table includes `role`, `status`, `last_login_at`
- Foreign keys follow Laravel conventions (`user_id`, `student_id`, `teacher_id`, etc.)

Migration order matters - run with `php artisan migrate` (never `migrate:fresh` in production without backup).

## Configuration

- Environment: `.env` file (never commit, copy from `.env.example`)
- Config files in `config/` (auth, database, mail, queue, etc.)
- App config: `config/app.php`
- Database config: Defaults to SQLite, supports MySQL/PostgreSQL

## Features to Note

- **Enrollment System**: Term-based enrollment with document requirements and fee management
- **Teacher Evaluations**: Student-submitted faculty evaluations
- **Document Requests**: Students can request official documents
- **Learning Resources**: Teachers can upload/manage course materials
- **Scheduler**: Room and schedule management
- **User Activity Logs**: SuperAdmin can monitor all user login activities, failed attempts, and security events with detailed device/browser information

## Development Notes

- Queue driver: Database (run `php artisan queue:listen` for background jobs)
- Session driver: Database
- Cache driver: Database
- Mail: Configured to `log` driver in development
- Sanctum installed for API token authentication (routes in `routes/api.php`)

---
> Source: [assassinationassi96-collab/Palale-High-School](https://github.com/assassinationassi96-collab/Palale-High-School) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
