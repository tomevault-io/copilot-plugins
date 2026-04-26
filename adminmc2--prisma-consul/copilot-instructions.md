## prisma-consul

> Marketing website + B2B discovery tool + document portal for PRISMA Consul, a pharma/healthcare consulting company. Self-hosted on IONOS VPS.

# PRISMA Consul - Web Platform

## Project Overview

Marketing website + B2B discovery tool + document portal for PRISMA Consul, a pharma/healthcare consulting company. Self-hosted on IONOS VPS.

**Live URL:** https://prismaconsul.com
**Hosting:** IONOS VPS (Ubuntu 24.04, nginx + Express.js + PM2)
**Database:** Neon PostgreSQL (serverless)
**File Storage:** Google Drive (via Workspace domain-wide delegation)

## Architecture

This is a monorepo with 3 frontend apps sharing one Express.js backend:

```
/                        → Marketing landing page (prismaconsul.com)
/apex                    → APEX Discovery Form (prismaconsul.com/apex) — requiere login
/hub                     → PRISMA Hub (prismaconsul.com/hub) — requiere login
/api/*                   → Express.js API backend
```

## Directory Structure

```
├── index.html                  # Landing page
├── aviso-legal.html            # Legal pages
├── cookies.html
├── privacidad.html
├── css/styles.css              # Landing page styles
├── js/main.js                  # Landing page scripts
├── images/                     # All media assets
│   ├── logos/                  # SVG logos, favicon
│   ├── team/                   # Team member photos
│   └── videos/                 # Marketing videos
├── apex/                       # APEX Discovery Form (self-contained SPA)
│   ├── index.html
│   ├── form.js                 # Main form logic (~3500 lines)
│   ├── form.css
│   ├── signal-detector.js
│   └── fonts/                  # Phosphor Icons (local)
├── portal/                     # PRISMA Hub (single-file SPA)
│   └── index.html              # Login + document management + admin panel
├── server/                     # Express.js backend
│   ├── server.js               # App setup, middleware, route mounting
│   ├── package.json            # All backend dependencies
│   ├── schema.sql              # PostgreSQL schema reference
│   ├── middleware/
│   │   ├── cors.js             # CORS headers (all routes)
│   │   └── auth.js             # JWT verification + requireAdmin middleware
│   ├── routes/
│   │   ├── portal.js           # Auth, upload, file management, user management, activity log
│   │   ├── apex.js             # Research, questions, form submission
│   │   └── ai.js               # Groq LLM chat + Whisper transcription
│   └── lib/
│       ├── pain-knowledge-base.js  # Pain/situation database (469 pains)
│       ├── google-drive.js     # Google Drive client + per-user folder helpers
│       └── fetch-timeout.js    # Fetch wrapper with AbortController
├── .env                        # Secrets (NOT committed)
├── .gitignore
└── .github/
    └── dependabot.yml          # Weekly dependency monitoring
```

## Tech Stack

- **Frontend:** Vanilla HTML/CSS/JS (no frameworks)
- **Fonts:** Quicksand (headings) + Source Sans 3 (body) via Google Fonts
- **Icons:** Phosphor Icons (local font files in `apex/fonts/`)
- **Backend:** Express.js with modular routes (server/routes/)
- **Database:** Neon PostgreSQL (`apex_submissions`, `portal_users`, `portal_files`, `portal_activity_log`)
- **Auth:** JWT (jsonwebtoken) + bcryptjs, 24h token expiry. Role-based: admin/user. Shared auth for APEX y Hub.
- **APIs:** Groq (LLM + Whisper), Tavily (web search), Claude API (questions)
- **File Storage:** Google Drive API via Service Account with domain-wide delegation
- **Email:** Gmail SMTP via Nodemailer

## Design System

- **Theme:** Dark (`--prisma-navy: #101B2C`, `--bg-primary: #1a2535`)
- **Accent:** `--tech-cyan: #31BEEF`
- **Secondary:** `--visionary-violet: #994E95`, `--soft-blue: #A1B8F2`
- **Text:** `--clinical-white: #FAF9F6`
- **Screen transitions:** opacity + translateY 400ms ease (APEX pattern)
- **Button styles:** `border-radius: 4px 25px 25px 4px` (primary), `25px 4px 4px 25px` (logout)

## Environment Variables

Required in `.env` on the VPS (`~/web-de-prisma/.env`):

```
# APIs
GROQ_API_KEY=
TAVILY_API_KEY=
CLAUDE_API_KEY=

# Database
DATABASE_URL=              # Neon PostgreSQL connection string

# Email
SMTP_USER=                 # Gmail address
SMTP_PASS=                 # Gmail app password

# Portal
PORTAL_SECRET=             # JWT signing secret
GOOGLE_SERVICE_ACCOUNT_KEY= # Full JSON of Google SA credentials
GOOGLE_DRIVE_FOLDER_ID=    # Target Drive folder ID
```

## Key Patterns

### APEX Form Flow
Login → Welcome → Company Input → Research (Tavily+Groq) → Swipe Cards → MaxDiff → Top4 → Phase 1 Questions → Phase 2 Adaptive → Pains → Audio → Contact → Thank You

### Business Type System
- `SITUACIONES_DISTRIBUIDOR` (16 cards, IDs: A-P) for distributors/pharma
- `SITUACIONES_CLINICA` (16 cards, IDs: CA-CP) for clinics
- Type detected from `research-company.js` field `detectado.es_clinica`

### PRISMA Hub (Portal)
- Login → Role detection → Admin gets 3 tabs (Documentos, Usuarios, Actividad), User gets 1 tab (Documentos)
- Upload: Drag files → Stage with type classification → Click "Subir" → Upload to user's Google Drive subfolder
- Admin can "view as user" to see any client's files
- Admin can create new users from the Usuarios tab

### Google Drive Integration
- Service Account with domain-wide delegation impersonates `info@prismaconsul.com`
- Scope: `https://www.googleapis.com/auth/drive`
- Client ID for delegation: `105667745242936760421`
- Per-user subfolders: `GOOGLE_DRIVE_FOLDER_ID/user_{id}/` — each user's files are isolated

## Database Tables

### `apex_submissions`
Stores APEX form completions with company data, research results, responses, pains, and audio.

### `portal_users`
Portal login users with: `id, email, password_hash, nombre, empresa, rfc, direccion, ciudad, cp, telefono, contacto_principal, cargo, sector, role, drive_folder_id, created_at, last_login`

### `portal_files`
File ownership tracking: `id, drive_file_id, user_id, file_name, file_size, mime_type, doc_type, created_at`

### `portal_activity_log`
Activity audit log: `id, user_id, action, details (JSONB), ip_address, created_at`
Actions: `login`, `upload`, `delete`, `rename`, `user_created`

## Portal Users

- **Admin:** info@prismaconsul.com (role: admin — dashboard completo con gestión de usuarios y actividad)
- **Client:** armc@prismaconsul.com (role: user — ARMC Aesthetic Rejuvenation Medical Center)

## Git Workflow

- **`main`** → Producción. El VPS despliega desde aquí. **NO trabajar directamente en main.**
- **`dev`** → Desarrollo. Todos los cambios se hacen aquí primero.
- **GitHub se mantiene** como repositorio central, backup del código e historial de cambios.

### Flujo de trabajo

1. Trabajar siempre en `dev`
2. Probar los cambios (local o en servidor de prueba)
3. Cuando esté verificado, fusionar a `main`: `git checkout main && git merge dev && git push`
4. Volver a `dev` para seguir: `git checkout dev`
5. Actualizar el servidor: `ssh prisma@212.227.251.125 "cd ~/web-de-prisma && git pull origin main"`

### Reglas

- Nunca hacer push directo a `main` sin haber probado en `dev`
- Antes de fusionar a `main`, verificar que todo funciona
- Los commits en `dev` pueden ser frecuentes y granulares
- Los merges a `main` deben representar cambios completos y funcionales
- Nunca editar código directamente en el servidor — siempre desde local

## Development

```bash
# Start local dev server (Express)
cd server && node server.js

# Access apps locally
http://localhost:3000          # Landing page
http://localhost:3000/apex     # APEX form
http://localhost:3000/documentacion  # Portal
```

## IONOS VPS (Producción)

**VPS:** IONOS, Ubuntu 24.04, IP `212.227.251.125`
**Acceso:** `ssh prisma@212.227.251.125` (clave SSH, sin contraseña)
**Credenciales locales:** `.server-credentials` (en .gitignore)
**Stack:** nginx + Express.js + PM2 + Let's Encrypt SSL
**Código en servidor:** `/home/prisma/web-de-prisma/` (rama `main`)
**Backup:** Acronis Cyber Protect (agente instalado, backup semanal completo + diario incremental)

### Securización (COMPLETADO)

1. Usuario `prisma` con sudo (root SSH desactivado)
2. Clave SSH ed25519 (contraseñas SSH desactivadas)
3. Firewall UFW (puertos 22, 80, 443, 44445)
4. Fail2ban activo
5. Actualizaciones automáticas (`unattended-upgrades`)
6. DNSSEC activado

### Stack del servidor (COMPLETADO)

1. Node.js 22 LTS + npm
2. nginx como reverse proxy (estáticos + `/api/*` → Express)
3. Express.js con rutas modulares (`server/server.js` + `server/routes/`)
4. PM2 gestionando Express (auto-restart, boot startup)
5. SSL con Let's Encrypt (certbot, renovación automática cada 90 días)
6. Variables de entorno en `~/web-de-prisma/.env`

### Pendiente

- **Deploy automático** — Configurar git hook o script para que al hacer push a `main` el servidor se actualice solo

## Comunicación

- Antes de ejecutar cada paso, explicar **qué se va a hacer, por qué y qué efecto tiene**
- Las explicaciones deben ser para un **perfil no especialista técnico**: lenguaje claro, analogías cuando sea útil, sin asumir conocimientos previos de infraestructura o devops
- Además de la explicación sencilla, incluir siempre una **explicación profesional**: qué es el programa/herramienta técnicamente, qué hace, por qué se ha elegido frente a otras alternativas y qué beneficios aporta
- Cada acción (en el servidor, en el código, en la configuración) debe ir acompañada de una **comprobación verificable** que confirme que el paso se completó correctamente
- Tras cada acción relevante o de impacto, **analizar si es necesario actualizar**: CLAUDE.md, la memoria del proyecto, el changelog, `.gitignore`, variables de entorno u otra documentación del proyecto. No esperar a que se acumulen cambios — documentar al momento

## Versionado

La versión actual se muestra en el footer de `index.html`. Se usa **Versionado Semántico** (`MAJOR.MINOR.PATCH`):
- **MAJOR** — Cambio grande: rediseño, nueva arquitectura (v1 → v2 → v3)
- **MINOR** — Funcionalidad nueva (v3.0 → v3.1)
- **PATCH** — Correcciones, bugs, parches de seguridad (v3.0.0 → v3.0.1)

**Versión actual:** `v3.2.13`

Al hacer cualquier cambio, actualizar la versión en:
1. El footer de `index.html` (línea del `footer__bottom`, en `data-es`, `data-en` y el texto visible)
2. La pantalla de login de `portal/index.html` (elemento `.welcome-version`)
3. La cabecera del `CHANGELOG.md` (nueva entrada con la versión)
4. Este campo "Versión actual" en CLAUDE.md

## Changelog

El archivo `CHANGELOG.md` en la raíz del proyecto registra todos los cambios relevantes. **Es obligatorio actualizarlo en cada cambio** que se haga al proyecto — ya sea código, configuración, infraestructura o dependencias. Cada entrada debe incluir:
- Fecha (`[YYYY-MM-DD]`)
- Categoría (Seguridad, Infraestructura, Frontend, Backend, Repositorio, etc.)
- Descripción clara del cambio y por qué se hizo

## Common Gotchas

- Phosphor Icons need BOTH classes: `ph ph-icon-name`
- Backend deps: install with `cd server && npm install`
- Google Drive SA needs domain-wide delegation in Google Admin console
- Spanish characters: use real UTF-8 chars, not `\u00xx` escapes in HTML
- SVG logo (`logo_simbolo_V2.svg`) has large viewBox whitespace — handle sizing in CSS, don't modify the SVG
- APEX y PRISMA Hub comparten autenticación (`server/middleware/auth.js` + tabla `portal_users`)
- Auth middleware exports `{ auth, requireAdmin }` — auth verifica JWT, requireAdmin exige role='admin'
- Google Drive: cada usuario tiene su subcarpeta (`user_{id}/`). Los archivos se rastrean en `portal_files`
- JWT incluye `{id, email, nombre, role}` — tokens antiguos sin role se tratan como 'user'

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adminmc2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
