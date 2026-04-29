## sysadmin-tools

> This is a Node.js web application built with TypeScript, Express, and EJS. It serves as an internal administration dashboard managing various modules including users, branches, terminals, services, and digital signage screens. The digital signage module integrates with Samsung TVs via Browser APIs and WebSockets, and with LG TVs via both DLNA (UPnP) and direct WebOS WebSocket connections. It utilizes Microsoft SQL Server for data storage and Socket.io/WebSockets for real-time communication and screen control.

# SysAdmin Forum

## Project Overview
This is a Node.js web application built with TypeScript, Express, and EJS. It serves as an internal administration dashboard managing various modules including users, branches, terminals, services, and digital signage screens. The digital signage module integrates with Samsung TVs via Browser APIs and WebSockets, and with LG TVs via both DLNA (UPnP) and direct WebOS WebSocket connections. It utilizes Microsoft SQL Server for data storage and Socket.io/WebSockets for real-time communication and screen control.

## Architecture & Technologies
- **Backend Framework:** Node.js, Express, TypeScript
- **Frontend:** EJS (Embedded JavaScript) templates, Vanilla JS/CSS (served from `public/`), Bootstrap, SweetAlert2
- **Database:** Microsoft SQL Server (`mssql`)
- **Authentication & Security:** JWT (JSON Web Tokens) with Cookies, role-based access control, and module-level permissions
- **Real-time & Network:** Socket.io, WebSockets (`ws`), DLNA/UPnP, Wake-on-LAN
- **Structure:** Modular MVC-like architecture organized by feature within `src/modules/` (e.g., `auth`, `dashboard`, `users`, `screens`).

## Development Conventions
- **Source Code:** All TypeScript source code is located in the `src/` directory.
- **Views:** EJS templates are located in `src/views/`. During the build process, they are copied to `dist/views/` via the `copy-views` script.
- **Static Assets:** CSS, JS, fonts, and media are served from the `public/` directory and are not processed by the build.
- **Modules:** Features are encapsulated in modules under `src/modules/`, typically containing a controller (`.controller.ts`), routes (`.routes.ts`), and service (`.service.ts`).
- **Middlewares:** Shared middlewares are found in `src/shared/middlewares/`, handling routing logic like authentication, role-based access, and specific module permissions.

## Building and Running

### Prerequisites
- Node.js (v18+ recommended)
- npm (or bun)
- SQL Server (database schema available in `sysadmin_db.sql` and `migrations/`)

### Development
```bash
# Install dependencies
npm install

# Run in development mode (uses nodemon, auto-reloads on .ts changes)
npm run dev
```

### Production Build
```bash
# Clean previous build (optional)
npm run clean

# Compile TypeScript to JavaScript and copy EJS views to the dist/ directory
npm run build

# Start the compiled application from dist/app.js
npm start
```

### Environment Variables
Create a `.env` file based on `.env.example`. Typical configuration variables include:
- `PORT` (Default is 3435)
- `JWT_SECRET`
- Database credentials (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`)

## Additional Notes
- **Deployment:** PM2 is recommended for production deployment (e.g., `pm2 start dist/app.js --name "sysadmin-forum"`).
- **Static Proxy:** The application includes an HTTP proxy setup in `src/app.ts` to bypass CORS and frame options for external content integration.

## Frontend & Design System (Nothing Style)

This project strictly adheres to a customized "Nothing Style" aesthetic for its UI, deeply modifying the default Bootstrap 5 framework.

### 1. Color Palette
- `--space-indigo` (`#2b2d42ff`): Primary dark backgrounds, cards, and panels.
- `--lavender-grey` (`#8d99aeff`): Secondary text, subtle borders, inactive states, and tags.
- `--platinum` (`#edf2f4ff`): Primary text and high-contrast highlights.
- `--punch-red` (`#ef233cff`): Hover states, secondary alerts, and glowing accents.
- `--flag-red` (`#d90429ff`): Primary buttons, critical states, and primary accents.

### 2. Typography
- **Body (`--font-body`):** *Space Grotesk* for clean, legible reading.
- **Headings & Accents (`--font-tech`):** *Silkscreen* (dot-matrix style) for `h1`-`h6`, card values, table headers, badges, and status tags.

### 3. UI/UX Rules
- **Glassmorphism:** Solid backgrounds are discouraged. Use `rgba` with `backdrop-filter: blur(12px)` for cards, modals, and dropdowns.
- **Orthogonal Design:** Minimal `border-radius` (2px-8px). Avoid soft, pill-shaped curves unless explicitly needed. No default Bootstrap `box-shadow`s; use glowing effects (`box-shadow` with primary colors) only for focus or hover states.
- **Inputs & Forms:** No blue focus rings. Use a single bottom border (`--lavender-grey`) that glows `--punch-red` on `:focus`. Placeholders must use the `--font-tech` font.
- **Tables:** Integrated into glassmorphic cards. Use `--font-tech` for headers, `--platinum` for cell data, and `--flag-red` for header separators.

### 4. JavaScript Architecture
- **Event Delegation:** NEVER use inline `onclick="..."` attributes in HTML/EJS files.
- **`data-action` pattern:** Bind actions using `data-action="myFunction()"` on buttons. The global event listener in `public/js/common.js` intercepts all clicks on the `document.body` and routes them, optimizing performance and handling dynamically rendered elements.

---
> Source: [JessZB/sysadmin-tools](https://github.com/JessZB/sysadmin-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
