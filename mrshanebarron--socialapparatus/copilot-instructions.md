## socialapparatus

> Modern social networking platform built with the TALL stack (Tailwind, Alpine, Laravel, Livewire).

# SocialApparatus - Project CLAUDE.md

## Overview
Modern social networking platform built with the TALL stack (Tailwind, Alpine, Laravel, Livewire).
Spiritual successor to Boonex Dolphin - self-hosted, open source, with web installer.

## URLs
- **Development**: http://community.test
- **Production Demo**: community.socialapparatus.com (future)
- **Marketing Site**: socialapparatus.com (future)

## Stack
- Laravel 12.x
- Livewire 3.x
- Alpine.js
- Tailwind CSS
- Jetstream (teams, dark mode, API)

## Database Strategy
- Installer allows choice: SQLite or MySQL
- SQLite default for quick start
- All migrations must be compatible with both

## Architecture

### Core Modules (Priority Order)

1. **Installer Wizard** (`/install`)
   - Check requirements (PHP, extensions)
   - Database configuration (SQLite/MySQL)
   - Admin account creation
   - Site settings (name, description)
   - Creates `.installed` lock file

2. **Users & Profiles**
   - Extended profile fields (customizable)
   - Avatar + cover photo
   - Privacy controls per field
   - Profile views tracking
   - Online status

3. **Connections System**
   - Friend requests (mutual)
   - Following (one-way)
   - Blocking
   - Connection suggestions

4. **Groups/Communities**
   - Public/Private/Secret types
   - Group roles (admin, moderator, member)
   - Group pages with activity feed
   - Group invitations

5. **Messaging**
   - Direct messages (1:1)
   - Group conversations
   - Real-time with Livewire polling or websockets
   - Message reactions

6. **Activity Feed**
   - Personal timeline
   - Network timeline (friends activity)
   - Site-wide public feed
   - Post types: text, photo, video, link, poll

7. **Media**
   - Photo albums
   - Video uploads (with transcoding queue)
   - Audio uploads
   - Media privacy controls

8. **Content Types**
   - Blog posts/Articles
   - Forums/Discussions
   - Events (with RSVP)
   - Marketplace/Classifieds
   - Polls

9. **Engagement**
   - Likes/Reactions
   - Comments (threaded)
   - Shares/Reposts
   - Mentions (@username)
   - Hashtags

10. **Monetization**
    - Membership levels (free, premium, etc.)
    - Stripe/PayPal integration
    - Points/Credits system
    - Paid content/subscriptions

11. **Admin Panel**
    - User management
    - Content moderation
    - Site settings
    - Analytics dashboard
    - Module enable/disable

12. **Theming**
    - Theme system (Tailwind-based)
    - Color scheme customization
    - Layout options

### Directory Structure
```
app/
├── Models/
│   ├── User.php (extended)
│   ├── Profile.php
│   ├── Connection.php
│   ├── Group.php
│   ├── Message.php
│   ├── Post.php
│   ├── Media.php
│   └── ...
├── Livewire/
│   ├── Installer/
│   ├── Profile/
│   ├── Feed/
│   ├── Groups/
│   ├── Messages/
│   └── ...
├── Services/
│   ├── InstallerService.php
│   ├── MediaService.php
│   ├── NotificationService.php
│   └── ...
└── ...
```

## Commands
- `php artisan serve` - Dev server (or use Herd)
- `php artisan migrate:fresh --seed` - Reset DB with seeds
- `npm run dev` - Vite dev server
- `npm run build` - Production build

## Installer Flow
1. `/install` - Welcome, check if already installed
2. `/install/requirements` - PHP version, extensions check
3. `/install/database` - Choose SQLite or MySQL, test connection
4. `/install/admin` - Create admin account
5. `/install/site` - Site name, description, settings
6. `/install/complete` - Finish, redirect to login

## Notes
- All features must work offline (no external API dependencies for core)
- Progressive enhancement - basic features work without JS
- Mobile-first responsive design
- Accessibility (WCAG 2.1 AA)

## Code Quality

### Static Analysis
- PHPStan with Larastan installed at level 5
- Baseline file: `phpstan-baseline.neon` (180 legacy errors)
- Run: `vendor/bin/phpstan analyse`
- IDE helper PHPDocs generated for all models

### Testing
- Run: `php artisan test`
- All 40 tests passing (4 skipped for disabled features)

### Code Style
- Laravel Pint configured
- Run: `vendor/bin/pint`

## Production Environment

### Required .env Settings
```env
# Security
APP_ENV=production
APP_DEBUG=false
SESSION_ENCRYPT=true
SESSION_SECURE_COOKIE=true

# Performance (use Redis)
CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
REDIS_HOST=127.0.0.1

# File Storage (use S3 or similar)
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-bucket
```

### Required Infrastructure
1. **Redis** - For caching, sessions, and queues
2. **Queue Worker** - Run with Supervisor:
   ```bash
   php artisan queue:work --sleep=3 --tries=3
   ```
3. **HTTPS** - All traffic must be HTTPS
4. **Scheduler** - Add to crontab:
   ```
   * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
   ```

### Security Checklist
- [ ] SESSION_ENCRYPT=true
- [ ] SESSION_SECURE_COOKIE=true
- [ ] All traffic redirected to HTTPS
- [ ] `composer audit` shows no vulnerabilities
- [ ] `npm audit` shows no vulnerabilities
- [ ] Review all Policies in app/Policies/

---
> Source: [mrshanebarron/socialapparatus](https://github.com/mrshanebarron/socialapparatus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
