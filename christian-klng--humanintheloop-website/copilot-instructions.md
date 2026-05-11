## humanintheloop-website

> Website for Human in the Loop. Plain HTML, CSS, and vanilla JavaScript served via nginx + Node.js API in a Docker container. A build-time Node.js script generates per-route HTML for OG meta tags; the API server regenerates them automatically after every admin change. An admin panel allows editing events and resources via the browser.

# Human in the Loop — Website

## Project Overview

Website for Human in the Loop. Plain HTML, CSS, and vanilla JavaScript served via nginx + Node.js API in a Docker container. A build-time Node.js script generates per-route HTML for OG meta tags; the API server regenerates them automatically after every admin change. An admin panel allows editing events and resources via the browser.

## Tech Stack

- HTML5, CSS3 (custom properties), vanilla JS
- Google Fonts (Switzer via Fontshare)
- Express.js API server (admin CRUD + data aggregation)
- Deployed via Coolify (Docker/nginx + Node.js) on https://humanintheloop.academy

## Project Structure

```
/
├── index.html              Main HTML template (with OG placeholders)
├── css/styles.css          All styles, design system variables
├── js/app.js               SPA router (History API), event rendering
├── js/admin.js             Admin panel UI logic
├── events/
│   └── events.json         Bundled event data (migration seed)
├── library/
│   └── resources.json      Bundled resource data (migration seed)
├── server/
│   ├── api.js              Express API server (auth + CRUD)
│   └── package.json        API server dependencies
├── scripts/
│   ├── generate-pages.js   Build-time OG meta tag generator
│   └── migrate-to-individual.js  Splits bundled JSON into individual files
├── nginx.conf              nginx routing + API proxy configuration
├── Dockerfile              Multi-stage: node build + nginx + API serve
├── docker-entrypoint.sh    Container startup (migration, OG gen, servers)
├── favicon.ico             Favicons and web manifest
├── site.webmanifest
├── corporate-design-system.md  Corporate Design System (colors, typography, print, etc.)
└── CLAUDE.md
```

## Corporate Design System

The file `corporate-design-system.md` contains the full Corporate Design System for Human in the Loop. It covers brand identity, logo usage, color system (HEX, RGB, CMYK), typography (Switzer), spacing, components, imagery guidelines, print specifications (business cards, flyers, ads, roll-ups), presentation templates, social media formats, and accessibility requirements. Refer to this document when creating any visual materials — digital or print.

## Naming

- **Never abbreviate "Human in the Loop"** — do not use "HITL" as it has unfortunate connotations in German. Always write the full name.

## Key Conventions

- **No CSS frameworks** — custom CSS with design tokens in `:root` variables
- **No inline styles** — all styling via classes in css/styles.css (exception: styleguide color swatches)
- **Events are data-driven** — individual JSON files on `/files/` volume, served via API
- **Library resources are data-driven** — individual JSON files on `/files/` volume, served via API
- **Media files served from `/files/` volume** — Docker volume mounted at `/files/`, referenced as `/files/library/...` in resource JSON
- **Path-based SPA routing** — URLs use `/`, `/events`, `/event/{id}`, `/library`, `/resource/{id}`, `/styleguide`, `/privacy`, `/terms`, `/imprint`, `/admin`
- **Semantic HTML** — use `<a>` and `<button>` (not `<div onclick>`), include ARIA labels
- **OG meta tags** — generated per-route at Docker build time via `scripts/generate-pages.js`; regenerated at container startup from volume data; also regenerated automatically by the API server after every admin write operation (create/update/delete); also updated client-side on navigation

## Color Palette

| Variable         | Hex       | Use                        |
|-----------------|-----------|----------------------------|
| `--accent`       | `#FFD166` | Primary buttons, accents   |
| `--secondary`    | `#073B4C` | Secondary buttons, bgs     |
| `--warning`      | `#EF476F` | Warnings, errors           |
| `--success`      | `#06D6A0` | Confirmations              |
| `--info`         | `#118AB2` | Notifications              |
| `--text-primary` | `#111111` | Main text                  |
| `--text-secondary`| `#8A8F98`| Muted text                 |

## Data Storage on `/files/` Volume

```
/files/
  ├── events/
  │   ├── event-slug.json           Individual event JSON files
  │   └── ...
  ├── library/
  │   ├── resource-slug/
  │   │   ├── resource.json         Resource data
  │   │   ├── thumb.jpg             Media files
  │   │   └── ...
  │   └── ...
  ├── uploads/                      Admin-uploaded media files
  │   ├── events/                 Event-related uploads
  │   ├── library/                Resource-related uploads
  │   └── ...
```

On first container startup, `migrate-to-individual.js` splits the bundled `events.json` and `resources.json` into individual files on the volume (idempotent).

## Admin Panel

- **Access**: Navigate to `/admin` (no link in public navigation)
- **Authentication**: Username + password login via `ADMIN_USER` and `ADMIN_PASSWORD` environment variables
- **Features**: Edit (raw JSON), add, and delete events and resources; upload media files (images/videos) with folder tabs, URL copy, and lightbox preview
- **API server**: Express.js on port 3000 (proxied by nginx at `/api/*`)
- **Session**: Bearer token stored in `sessionStorage`, 24h expiry

### Admin Routes

| Route | View |
|-------|------|
| `/admin` | Login form |
| `/admin/dashboard` | Dashboard listing events & resources |
| `/admin/event/{id}` | JSON editor for event |
| `/admin/resource/{id}` | JSON editor for resource |
| `/admin/new/event` | Create new event |
| `/admin/new/resource` | Create new resource |

### API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/login` | No | Authenticate, returns token |
| `POST` | `/api/logout` | Yes | Invalidate token |
| `GET` | `/api/auth/check` | Yes | Verify token validity |
| `GET` | `/api/events` | No | List all events |
| `GET` | `/api/events/:id` | No | Single event |
| `PUT` | `/api/events/:id` | Yes | Update event |
| `POST` | `/api/events` | Yes | Create event |
| `DELETE` | `/api/events/:id` | Yes | Delete event |
| `GET` | `/api/resources` | No | List all resources |
| `GET` | `/api/resources/:id` | No | Single resource |
| `PUT` | `/api/resources/:id` | Yes | Update resource |
| `POST` | `/api/resources` | Yes | Create resource |
| `DELETE` | `/api/resources/:id` | Yes | Delete resource |
| `GET` | `/api/uploads?folder=` | Yes | List uploaded media files (folder: `events`, `library`, or empty for root) |
| `POST` | `/api/uploads?folder=` | Yes | Upload file (multipart/form-data, max 50 MB) |
| `DELETE` | `/api/uploads/:filename?folder=` | Yes | Delete uploaded file |

## Adding a New Resource

**Via admin panel** (preferred): Navigate to `/admin` → login → Add Resource → edit JSON → Save.

**Via files**: Create `/files/library/{resource-id}/resource.json` on the Docker volume.

### Resource JSON Schema

```json
{
    "id": "url-safe-slug",
    "title": "Resource Title",
    "date": "2026-02-20",
    "author": "Author Name",
    "description": ["Paragraph 1", "Paragraph 2"],
    "tags": ["Tag1", "Tag2"],
    "thumbnail": "/files/library/slug/thumb.jpg",
    "images": [
        { "src": "/files/library/slug/photo.jpg", "alt": "Description", "caption": "Optional caption" }
    ],
    "video": { "type": "html5|youtube|vimeo", "src": "path-or-embed-id", "poster": "optional-poster.jpg" }
}
```

- `images` can be `[]` if no gallery; `video` can be `null` if no video
- `video.type`: `html5` (self-hosted, `src` is file path), `youtube`/`vimeo` (`src` is embed ID)
- Resources are sorted by date (newest first) automatically

## Adding a New Event

**Via admin panel** (preferred): Navigate to `/admin` → login → Add Event → edit JSON → Save.

**Via files**: Create `/files/events/{event-id}.json` on the Docker volume.

### Event JSON Schema

```json
{
    "id": "url-safe-slug",
    "title": "Event Title",
    "date": "12. Okt. 2026",
    "dateFull": "12. Oktober 2026",
    "time": "10:00 – 15:00 Uhr (MEZ)",
    "location": "Livestream",
    "locationNote": "Link nach Anmeldung",
    "type": "Livestream",
    "cost": "Kostenlos für Alumni",
    "spots": "maximal 6",
    "pricing": "free",
    "stripeLink": null,
    "onlineLink": "https://zoom.us/j/...",
    "confirmationText": "Dein Platz ist bestätigt! Du erhältst den Link kurz vor der Session.",
    "tags": ["Tag1", "Tag2"],
    "image": "/files/uploads/events/event-photo.jpg",
    "description": ["Paragraph 1", "Paragraph 2"],
    "learns": ["Point 1", "Point 2"],
    "audience": "Target audience description."
}
```

- `pricing`: `"free"` or `"paid"` (Stripe redirect)
- `stripeLink`: Full Stripe Payment Link URL for paid events (one link per event series), `null` for free events
- `onlineLink`: Event URL (Zoom etc.), `null` if not applicable
- `confirmationText`: Custom text for confirmation messages

## Event Registration

Events support two registration modes controlled by the `pricing` field:

**Free events** (`pricing: "free"`): Currently shows a registration form, but the webhook backend is not connected. The `__WEBHOOK_URL__` placeholder in `index.html` remains unreplaced, and `app.js` detects this and shows a "not available" message.

**Paid events** (`pricing: "paid"`): User enters email → Redirect to Stripe Payment Link with `client_reference_id={event.id}` and `prefilled_email={email}`. One Stripe Payment Link is created per event series; `client_reference_id` identifies the specific event instance.

## Local Development

```sh
npx serve -s -l 8000
# SPA-aware static server — serves index.html for all routes
# Note: API endpoints require the Node.js server (see Docker instructions)
```

## Docker Build

```sh
docker build --build-arg BASE_URL=https://humanintheloop.academy -t humanintheloop .
docker run -p 8080:80 -e ADMIN_USER=admin -e ADMIN_PASSWORD=yourpassword -v ./test-files:/files humanintheloop
```

`BASE_URL` is required (build arg) — it sets the absolute URLs for OG meta tags and canonical links.
`ADMIN_USER` is required (env var) — it sets the admin login username.
`ADMIN_PASSWORD` is required (env var) — it sets the admin login password.

---
> Source: [christian-klng/humanintheloop_website](https://github.com/christian-klng/humanintheloop_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
