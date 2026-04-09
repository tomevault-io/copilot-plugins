## daycare-time-keep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Application

**Start the application:**
```bash
docker-compose up -d
```

**Stop the application:**
```bash
docker-compose down
```

**Rebuild after code changes:**
```bash
docker-compose up -d --build
```

**Access points:**
- Parent interface: http://localhost:8080/index.php
- Staff dashboard: http://localhost:8080/admin.php
- Reports: http://localhost:8080/reports.php

**Default test credentials (PIN: 123456):**
- Test family with 2 children
- Admin staff account

## Architecture Overview

### Technology Stack
- **Backend:** Pure PHP 8.2 (no frameworks)
- **Frontend:** HTML5, CSS3, Vanilla JavaScript (no frameworks)
- **Storage:** File-based JSON (no SQL database)
- **Server:** Apache with mod_rewrite
- **Deployment:** Docker (php:8.2-apache)

### Directory Structure Philosophy

```
/
├── config.php              # Central config & utilities - loaded by ALL files
├── src/                    # Business logic (NOT web-accessible)
│   ├── auth.php           # Session-based authentication
│   └── timekeeping.php    # Core check-in/out logic
├── public/                # Web root (Apache DocumentRoot)
│   ├── index.php          # Parent interface
│   ├── admin.php          # Staff dashboard (largest file: 1505 lines)
│   └── reports.php        # Reporting interface
└── data/                  # JSON data storage (persisted via Docker volume)
    ├── families.json      # Family & children (includes base64 photos)
    ├── users.json         # Staff credentials
    ├── settings.json      # Configurable business rules
    └── timekeeping/       # Monthly partitioned attendance records
        └── YYYY-MM.json   # One file per month
```

**Key Architectural Decision:** The `src/` directory contains business logic that should NEVER be directly web-accessible. Apache's DocumentRoot points to `public/` only.

### Data Flow Patterns

**Check-In/Out Process:**
1. Parent logs in with PIN → `auth.php` validates against hashed PIN in `families.json`
2. PHP session stores family ID
3. `index.php` displays children from `families.json`
4. Check-in → `timekeeping.php::checkInChild()` creates record in current month's JSON
5. Check-out → calculates duration, overtime charges, late pickup fees
6. Updates same record with checkout time and all fees

**File-Based Transactions:**
- Read entire JSON file into memory (via `loadJsonFile()`)
- Modify PHP array
- Write entire file back atomically (via `saveJsonFile()`)
- No locking mechanism (acceptable for small daycare scale)

### Configurable Business Rules

All business rules are stored in `data/settings.json` and loaded at runtime in `config.php`:

```php
// These constants are loaded from settings.json:
MAX_HOURS_PER_DAY          // Default: 8.5 hours
OVERAGE_RATE_PER_MINUTE    // Default: $1.00
DAYCARE_CLOSING_TIME       // Default: 16:30:00 (4:30 PM)
LATE_PICKUP_RATE_PER_MINUTE // Default: $1.00
```

Staff can edit these via Admin → Settings tab. Changes take effect immediately on page refresh.

### Dual Fee System

The application calculates TWO separate fee types:

1. **Overtime Fee:** Applied when total duration exceeds `MAX_HOURS_PER_DAY`
   - Example: Child stays 9 hours when max is 8.5 → 30 minutes × $1/min = $30

2. **Late Pickup Fee:** Applied when checkout time is after `DAYCARE_CLOSING_TIME`
   - Example: Checkout at 5:00 PM when closing is 4:30 PM → 30 minutes × $1/min = $30

**Both can apply to the same checkout.** Records store both separately:
```json
{
  "overage_minutes": 30,
  "overage_charge": 30.00,
  "late_pickup_minutes": 30,
  "late_pickup_charge": 30.00
}
```

## Common Development Tasks

### Adding New Configuration Setting

1. Add to `data/settings.json` with default value
2. Update `config.php` defaults array (lines 35-42)
3. Define constant from setting (lines 48-52)
4. Add form field in `admin.php` Settings tab (lines 1244-1348)
5. Update handler in `admin.php` (lines 405-434)

### Modifying Fee Calculations

All fee calculations happen in `src/timekeeping.php`:
- `checkOutChild()` function (lines 142-214) - calculates fees at checkout
- `edit_times` handler in `admin.php` (lines 357-418) - recalculates when editing
- `add_record` handler in `admin.php` (lines 489-531) - calculates for manually added records

**Important:** When changing fee logic, update ALL three locations to maintain consistency.

### Working with JSON Data

**Helper Functions (all in `config.php`):**
```php
loadJsonFile($filepath)    // Read JSON → PHP array
saveJsonFile($filepath, $data) // Write PHP array → JSON
loadSettings()             // Get settings array
saveSettings($settings)    // Update settings.json
```

**Data Schema Reference:**

Family structure in `families.json`:
```php
[
  'id' => 'fam_timestamp_random',
  'name' => 'Family Name',
  'pin_hash' => '$2y$10$...',  // bcrypt hash
  'contact_phone' => '555-0123',
  'contact_email' => 'email@example.com',
  'children' => [
    [
      'id' => 'child_timestamp_random',
      'first_name' => 'John',
      'last_name' => 'Doe',
      'birth_date' => 'YYYY-MM-DD',
      'photo' => 'data:image/png;base64,...',  // Base64 encoded
      'notes' => 'Allergies, etc.'
    ]
  ]
]
```

Timekeeping record structure:
```php
[
  'id' => 'rec_timestamp_random',
  'child_id' => 'child_...',
  'date' => 'YYYY-MM-DD',
  'check_in_time' => 'HH:MM:SS',
  'check_out_time' => 'HH:MM:SS',
  'duration_hours' => 8.5,
  'overage_minutes' => 30,
  'overage_charge' => 30.00,
  'late_pickup_minutes' => 0,
  'late_pickup_charge' => 0.00,
  'checked_in_by' => 'parent',
  'checked_out_by' => 'parent',
  'notes' => ''
]
```

### Authentication System

Two separate authentication systems:

**Family Authentication:**
- PIN-based (4-6 digits)
- Hashed with bcrypt (`password_hash()`)
- Session key: `$_SESSION['family_id']`
- Functions: `loginParent()`, `isParentLoggedIn()`, `getCurrentFamily()`

**Staff Authentication:**
- PIN-based (4-6 digits)
- Role-based access (staff/admin)
- Session key: `$_SESSION['staff_id']`
- Functions: `loginStaff()`, `isStaffLoggedIn()`, `requireStaffLogin()`, `getCurrentStaff()`

All auth functions in `src/auth.php`.

### Admin Dashboard Tab System

`admin.php` uses JavaScript tabs without page reloads:
- Families, Staff, Edit Times, Settings tabs are `<div>` containers
- Reports is a direct `<a href="/reports.php">` link
- Tab switching via `showTab(tabName)` JavaScript function
- URL parameters can force specific tab (e.g., `?date=2025-11-13` shows Edit Times tab)

When adding features to admin dashboard:
1. Add tab button to navigation (line 695)
2. Create tab content div with `id="tab-yourname"` and `class="tab-content hidden"`
3. Update JavaScript to handle new tab if needed

## Important Technical Notes

### Photo Storage
Child photos are stored as **base64-encoded data URIs** directly in `families.json`. This explains why the file can be 2MB+. Photos are displayed inline:
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANS..." />
```

### Monthly Partitioning
Timekeeping records are split into monthly files to prevent a single massive JSON file:
- Format: `data/timekeeping/YYYY-MM.json`
- Created automatically when first record for month is added
- Reports can span multiple months by loading multiple files

### Security Headers
`.htaccess` in `public/` sets security headers:
```apache
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
```

### ID Generation Pattern
All IDs use `timestamp_randomnumber` format:
```php
generateId('child_') → 'child_1699999999_4567'
```
This ensures uniqueness without auto-increment counters.

### No External Dependencies
The application intentionally has:
- No Composer dependencies
- No npm/Node.js requirements
- No external CSS/JS libraries
- Pure vanilla PHP/JavaScript

This makes deployment extremely simple but means you implement everything from scratch.

## Codebase Philosophy

This codebase is **intentionally educational** with:
- Extensive inline comments explaining every step
- Function-level docblocks
- Learning-focused documentation (GETTING_STARTED.md, CHEAT_SHEET.md)
- Simple patterns over advanced abstractions

When making changes, maintain this beginner-friendly style:
- Add comments explaining "why" not just "what"
- Use descriptive variable names
- Avoid clever one-liners
- Prefer clarity over brevity

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ubun2Junky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ubun2Junky)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
