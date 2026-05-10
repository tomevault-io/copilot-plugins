## diluition-calc

> **Calcolo Diluizioni** (Dilution Calculator) is a single-page web application for calculating product dilution ratios, primarily targeting car care and detailing products. The app is written entirely in Italian.

# CLAUDE.md - AI Assistant Guidelines for Calcolo Diluizioni

## Project Overview

**Calcolo Diluizioni** (Dilution Calculator) is a single-page web application for calculating product dilution ratios, primarily targeting car care and detailing products. The app is written entirely in Italian.

## Quick Start

The frontend (`diluizioni.html`) talks to a PHP backend (`api.php`) that reads
and writes `products.json`, so a PHP-capable server is required — a plain
static server is no longer enough.

```bash
# From the repo root
php -S localhost:8000
# Then open http://localhost:8000/diluizioni.html
```

For the admin area to work in local dev, create a `config.php` with a bcrypt
hash (the file is git-ignored):

```bash
php -r "echo \"<?php\nconst ADMIN_PASSWORD_HASH = '\" . password_hash('admin', PASSWORD_BCRYPT) . \"';\n\";" > config.php
```

## Project Structure

```
calcolo-diluizioni/
├── CLAUDE.md            # This file - AI assistant guidelines
├── README.md            # Project description (Italian)
├── diluizioni.html      # Frontend SPA (HTML + CSS + JS in one file)
├── api.php              # Backend REST-ish API (products CRUD + admin auth)
├── products.json        # Persistent product catalog (written by api.php)
├── config.php           # [git-ignored] admin password hash
├── setup.php            # [git-ignored] one-shot admin password setup helper
└── lxc/                 # LXC deployment documentation
    ├── README.md        # Production snapshot + redeploy runbook
    └── nginx-site.conf  # Versioned copy of the production Nginx site
```

### File Organization (diluizioni.html)

| Section | Lines | Description |
|---------|-------|-------------|
| HTML Head & Meta | 1-6 | Document declaration, UTF-8, viewport |
| CSS Styles | 7-320 | Complete inline stylesheet |
| HTML Structure | 321-445 | Tab-based UI with three sections |
| JavaScript | 447-781 | Application logic and data |
| Closing Tags | 782-783 | `</body>` and `</html>` |

## Architecture

### Technology Stack

- **Frontend**: HTML5 + CSS3 + Vanilla ES6+ in a single file (`diluizioni.html`)
- **Backend**: PHP 8.x (`api.php`) — stateless except for a PHP session used for admin auth
- **Persistence**: flat JSON file (`products.json`) written by `api.php`
- **Web server**: Nginx + PHP-FPM (production, LXC container)
- **No build process**: the frontend is served as-is, no bundler, no transpiler

### Key Design Decisions

1. **Frontend in one file**: all HTML, CSS, and JS in `diluizioni.html` for simplicity
2. **Minimal backend**: `api.php` is a single PHP file with a handful of REST-ish endpoints; no framework
3. **No database**: products live in `products.json` on disk, read/written by `api.php`. Good enough for the scale of the app.
4. **Mobile-first**: Container max-width 480px with responsive adjustments
5. **State-driven UI**: Simple state variables (`selectedVolume`, `selectedRatio`, etc.)

## Core Features

### 1. Calcolatore (Calculator Tab)
- Main dilution calculator using ratio format 1:X
- Preset buttons for common volumes and ratios
- Auto-calculation when presets selected

### 2. Rabbocco (Refill Tab)
- Calculates how to refill bottles with existing diluted product
- Validates feasibility and shows appropriate error messages
- Tracks existing concentrate in bottle

### 3. Riferimento Prodotti (Products Tab)
- Reference guide for the supported cleaning products
- Each product has multiple usage scenarios with recommended ratios
- Clicking a use auto-sets the ratio and switches to Calculator tab
- The list is loaded at startup via `GET /api.php` (reads `products.json`)

### 4. Area Admin (password protected)
- Login via `POST /api.php?action=login` (bcrypt password check in `config.php`)
- Session-based auth (PHP `$_SESSION['admin']`)
- CRUD operations on products: add / update / delete, persisted in `products.json`
- UI exposed inside `diluizioni.html` when the user is authenticated

## Code Conventions

### Naming

- **Functions**: camelCase (`calculateRefill`, `renderVolumePresets`)
- **Variables**: camelCase (`selectedVolume`, `volumePresets`)
- **CSS Classes**: kebab-case (`tab-content`, `btn-primary`, `result-row`)
- **IDs**: kebab-case (`calc-result`, `product-select`)

### JavaScript Patterns

```javascript
// State management - simple variables
let selectedVolume = '';
let selectedRatio = '';

// Render functions - return HTML strings via template literals
function renderVolumePresets() {
  container.innerHTML = volumePresets.map(v =>
    `<button class="preset-btn" onclick="selectVolume(${v})">${formatVolume(v)}</button>`
  ).join('');
}

// Event handlers - inline onclick for presets, addEventListener for inputs
document.getElementById('volume').addEventListener('input', function() { ... });
```

### CSS Color System (Tailwind-inspired)

- Primary Blue: `#2563eb`
- Cyan: `#0891b2`
- Green: `#16a34a`
- Red: `#dc2626`
- Gray scale: `#f9fafb` to `#1f2937`

## Key Calculation Formulas

### Basic Dilution (1:X ratio)
```javascript
// For ratio 1:X, total parts = 1 + X
totalParts = 1 + ratio;
product = volume / totalParts;
water = product * ratio;
```

### Refill Calculation
```javascript
existingConcentrate = remaining / (1 + currentRatio);
totalConcentrateNeeded = bottle / (1 + targetRatio);
concentrateToAdd = totalConcentrateNeeded - existingConcentrate;
volumeToAdd = bottle - remaining;
waterToAdd = volumeToAdd - concentrateToAdd;
```

## Common Development Tasks

### Adding a New Product

Add to the `products` array (lines 453-533):

```javascript
{
  name: "Product Name",
  category: "Category",
  uses: [
    { name: "Use case 1", ratio: 10 },
    { name: "Use case 2", ratio: "5-10", ratioValue: 5 }, // Range with default
  ]
}
```

Notes:
- `ratio` can be a number or string (for ranges like "5-10")
- When using a range string, provide `ratioValue` for the default click value

### Adding a Preset Ratio

Add to `ratioPresets` array (line 450):
```javascript
const ratioPresets = [1, 2, 3, ..., 1200, YOUR_NEW_RATIO];
```

### Adding a Preset Volume

Add to `volumePresets` array (line 449):
```javascript
const volumePresets = [500, 1000, 2000, ..., YOUR_NEW_VOLUME];
```

### Adding a Bottle Preset

Add to `bottlePresets` array (line 451):
```javascript
const bottlePresets = [500, 750, 1000, YOUR_NEW_BOTTLE];
```

### Modifying Calculations

- Basic dilution: `calculate()` function (lines 635-655)
- Refill logic: `calculateRefill()` function (lines 658-718)

### Styling Changes

All CSS is inline in the `<style>` block (lines 7-320). Key classes:
- `.card` - Main container
- `.tab` / `.tab-content` - Tab system
- `.preset-btn` - Preset buttons
- `.result` - Result display boxes
- `.btn-primary` - Main action button

## Data Structures

### Product Schema

Matches the shape written by `api.php` into `products.json`. Each product has
a stable `id` generated server-side on creation:

```javascript
{
  id: number,             // Numeric ID, auto-incremented by api.php
  brand: string,          // Brand (e.g., "Cleantle", "Labocosmetica")
  name: string,           // Product name
  category: string,       // Category (e.g., "APC", "Shampoo")
  note: string | null,    // Optional free-text note
  uses: [{
    name: string,         // Use case description
    ratio: number|string, // Ratio value or range
    ratioValue?: number   // Optional: default value for ranges
  }]
}
```

### Presets
```javascript
volumePresets = [500, 1000, 2000, 5000, 8000, 10000, 12000];     // ml
ratioPresets = [1,2,3,...,10,15,20,...,100,200,300,400,800,1000,1200]; // 29 values
bottlePresets = [500, 750, 1000];                                  // ml (refill tab)
```

### Current Products

The authoritative list is in `products.json` and can drift at runtime because
it is editable via the admin UI. As of the last seed: 1 Cleantle + 7
Labocosmetica products. Don't rely on this list in code — always read from
`products.json` / `GET /api.php`.

### Key Functions Reference

| Function | Line | Purpose |
|----------|------|---------|
| `init()` | 542 | App initialization, renders all presets and sets up tabs |
| `formatVolume(ml)` | 550 | Formats ml to "Xml" or "XL" display |
| `renderVolumePresets()` | 555 | Renders volume preset buttons |
| `renderRatioPresets()` | 562 | Renders ratio preset buttons |
| `renderBottlePresets()` | 569 | Renders bottle preset buttons |
| `renderProductSelect()` | 576 | Populates product dropdown |
| `setupTabs()` | 583 | Wires up tab click handlers |
| `selectVolume(v)` | 595 | Handles volume preset selection |
| `selectRatio(r)` | 602 | Handles ratio preset selection |
| `selectBottle(b)` | 610 | Handles bottle preset selection |
| `calculate()` | 635 | Main dilution calculation |
| `calculateRefill()` | 658 | Refill/top-up calculation |
| `onProductChange(e)` | 721 | Product dropdown change handler |
| `selectProductUse(name, ratioValue)` | 742 | Sets ratio from product use and switches to calculator |

## Backend API (`api.php`)

A single-file PHP endpoint. Routing is based on the HTTP method plus a
`?action=...` query parameter. Session-based auth: the admin login sets
`$_SESSION['admin'] = true` and subsequent write operations call
`requireAuth()`.

| Method | Action      | Auth  | Purpose |
|--------|-------------|-------|---------|
| GET    | (none)      | none  | Returns the raw content of `products.json` |
| GET    | `check`     | none  | Returns `{authenticated: bool}` for the current session |
| POST   | `login`     | none  | Body `{password}` → verifies against `ADMIN_PASSWORD_HASH` from `config.php` |
| POST   | `logout`    | none  | Destroys the session |
| POST   | `add`       | admin | Body `{brand, name, category, note?, uses}` → creates a new product, assigns `id` |
| PUT    | (none)      | admin | Body `{id, brand, name, category, note?, uses}` → updates an existing product |
| DELETE | (none)      | admin | Body `{id}` → removes a product |

Notes:
- `config.php` is git-ignored and must define `const ADMIN_PASSWORD_HASH = '...'`
  (bcrypt, produced via `password_hash(..., PASSWORD_BCRYPT)`).
- If `ADMIN_PASSWORD_HASH` is empty, `login` returns `503 "Password admin non impostata"`.
- Writes go through `writeProducts()` which uses `file_put_contents` with
  `JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES`. No
  locking — fine for the low-traffic, single-admin usage pattern.
- When running in production, `products.json` must be writable by the PHP-FPM
  user (e.g. `www-data:www-data` with `0664`).

## Deployment

The app runs inside a **non-privileged LXC container on Proxmox VE**
(Debian 13 Trixie + Nginx + PHP-FPM 8.4). The container was provisioned
manually — there are no bootstrap scripts in the repo. What lives in
[`lxc/`](lxc/) is documentation and one versioned config file:

- `lxc/README.md` — production snapshot + redeploy runbook + admin password
  management + troubleshooting
- `lxc/nginx-site.conf` — versioned copy of the Nginx site deployed at
  `/etc/nginx/sites-available/calcolo-diluizioni` (blocks `config.php`,
  `.env*`, dotfiles; `fastcgi_pass` to `/run/php/php-fpm.sock`)

The previous FTP-based deploy to activecloud (via GitHub Actions) has been
removed. If a change breaks production, **do not** restore the old FTP workflow;
diagnose the LXC container instead.

Production layout inside the container:

```
/var/www/calcolo-diluizioni/
├── diluizioni.html      # 0644 www-data:www-data
├── api.php              # 0644 www-data:www-data
├── products.json        # 0664 www-data:www-data (writable by PHP-FPM)
└── config.php           # 0640 root:www-data     (readable by PHP-FPM only)
```

Redeploy flow (inside the container, after a merge to `main`):

```bash
cd /tmp/diluition-calc && git fetch origin main && git reset --hard origin/main
install -m 0644 -o www-data -g www-data diluizioni.html /var/www/calcolo-diluizioni/diluizioni.html
install -m 0644 -o www-data -g www-data api.php        /var/www/calcolo-diluizioni/api.php
systemctl reload php8.4-fpm
```

See `lxc/README.md` for the full runbook.

## Testing

No automated tests exist. Manual testing procedure:
1. Run `php -S localhost:8000` from the repo root and open `http://localhost:8000/diluizioni.html`
2. Test each tab's functionality
3. Verify calculations with known values
4. Test edge cases (empty inputs, impossible dilutions)
5. Test on mobile viewport
6. If touching admin/API code: test login + add/edit/delete product flows

Example verification:
- 1L (1000ml) with ratio 1:10 should yield:
  - Product: 90.91 ml
  - Water: 909.09 ml

## Language

The application is entirely in **Italian**. Key terms:
- Calcolatore = Calculator
- Diluizione = Dilution
- Rapporto = Ratio
- Rabbocco = Refill
- Prodotto = Product
- Acqua = Water
- Flacone = Bottle
- Quantità = Quantity

## Browser Support

- Modern evergreen browsers (Chrome, Firefox, Safari, Edge)
- Mobile-responsive (viewport meta tag included)
- Requires JavaScript enabled

## Important Notes for AI Assistants

1. **Frontend in one file**: All frontend changes go in `diluizioni.html`
2. **Backend in one file**: All API changes go in `api.php`
3. **No build step**: Changes are immediately testable by refreshing the browser (after `php -S` in local dev)
4. **Italian language**: Maintain Italian for all user-facing text
5. **Keep it simple**: No external dependencies, no frameworks (on either frontend or backend)
6. **Mobile-first**: Test any UI changes at narrow viewport widths
7. **Preserve structure**: Keep CSS/HTML/JS sections in their current order in `diluizioni.html`
8. **Calculation accuracy**: Double-check math formulas — users rely on these for real measurements
9. **Products are dynamic**: Don't hardcode product data in the frontend; read from `GET /api.php`
10. **Never commit secrets**: `config.php` and `.env` are git-ignored and must stay that way

## Git Workflow

- `main` branch contains production code
- Feature branches for new development (commonly `claude/...` for AI-assisted work)
- Commit messages can be in Italian or English
- No automated CI/CD: deployment to the LXC container is triggered manually via `lxc/deploy.sh`
- The old FTP-based GitHub Actions workflow has been removed

---
> Source: [alexpani/diluition-calc](https://github.com/alexpani/diluition-calc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
