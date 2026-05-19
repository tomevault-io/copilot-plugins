## life-of-josh

> **Location:** `~/life-of-josh/`

# LIFE OF JOSH APP - CLAUDE CONTEXT FILE

**Location:** `~/life-of-josh/`
**Port:** 3030
**URL:** http://localhost:3030

## QUICK REFERENCE - FILE STRUCTURE

```
~/life-of-josh/
├── CLAUDE.md           # THIS FILE - read first
├── package.json        # Node.js dependencies
├── src/
│   ├── server.js       # EXPRESS BACKEND (all API endpoints) ~2500 lines
│   └── routes/
│       ├── ai.js       # AI/Olivia chat routes
│       └── mcp.js      # MCP server routes
└── public/
    ├── index.html      # FRONTEND (all HTML/CSS/JS) ~2200 lines
    ├── manifest.json   # PWA manifest
    ├── sw.js           # Service worker for offline
    └── icons/          # App icons
```

## KEY SECTIONS TO EDIT

### Backend (server.js)
| Line Range | Section | What It Does |
|------------|---------|--------------|
| 1-60 | Config | Express setup, paths, IPs |
| 60-110 | Olivia API | AI chat integration |
| 110-270 | Weather & Scheduler | Weather API, smart task optimization |
| 270-420 | Grant Tracker | Grant discovery and tracking |
| 420-600 | Config/Service Lists | DOCKER_SERVICES, LOCAL_SERVICES, EXTERNAL_TOOLS arrays |
| 600-720 | Services API | /api/services, /api/n8n/workflows, /api/webhook-router |
| 720-820 | Tools & Repos API | /api/tools, /api/repos |
| 820-960 | **Projects API** | /api/projects - scans ~/repos and ~/phoenix-sync/PROJECTS |
| 960-1100 | Todo System | /api/todos - reads from ~/phoenix-sync/todo-system/categories/ |
| 1100-1400 | AI Tools | /api/ai/status, tool availability |
| 1400-1700 | File Browser | /api/files, /api/shortcuts |
| 1700-1900 | Scripts | /api/scripts/all, /api/scripts/run |
| 1900-2100 | System | /api/system, /api/processes, /api/ports |
| 2100-2300 | Repos & Weather | /api/repos/all, /api/weather |
| 2300-2450 | Hardware | /api/klipper/status, /api/car/status |
| 2450-2550 | Cron & Server | /api/cron, server startup |

### Frontend (public/index.html)
| Line Range | Section | What It Does |
|------------|---------|--------------|
| 1-200 | CSS | All styling (dark theme, cards, badges) |
| 200-220 | Nav | Section buttons (dashboard, olivia, todos, projects, etc.) |
| 220-600 | HTML Sections | Static section containers |
| 600-750 | Dashboard JS | loadDashboard() |
| 750-920 | Todos JS | loadTodos(), showTodoDetails() |
| 920-1010 | Services JS | loadServices() |
| 1010-1110 | **Projects JS** | loadProjects(), viewHardwareProject() |
| 1110-1170 | Docs/Files JS | loadDocs(), loadFiles() |
| 1170-1270 | Calendar/Ezra JS | loadCalendar(), loadEzra() |
| 1270-1460 | Olivia/Scheduler JS | sendToOlivia(), loadScheduler() |
| 1460-1700 | Grants JS | loadGrants(), filterGrants() |
| 1700-1900 | Commands/Voice JS | Command palette, voice commands |
| 1900-2110 | Hardware JS | loadTailscaleDevices(), loadKlipperStatus(), loadCarStatus() |

## API ENDPOINTS SUMMARY

### Core
- `GET /api/dashboard` - Summary stats
- `GET /api/projects` - All projects (repos + hardware)
- `GET /api/todos` - All todo categories
- `GET /api/docs` - Documentation files

### AI
- `GET /api/ai/status` - AI tools availability
- `POST /api/olivia/chat` - Chat with Olivia AI

### Hardware
- `GET /api/klipper/status` - 3D printer status
- `GET /api/car/status` - Car dashboard status
- `GET /api/tailscale/devices` - Network devices

### Files
- `GET /api/files?path=` - Browse directory
- `GET /api/files/read?path=` - Read file content

## DATA SOURCES

### Todo System
Location: `~/phoenix-sync/todo-system/categories/*.json`
Each JSON has:
```json
{
  "_meta": { "code": "...", "name": "...", "icon": "...", "tagline": "...", "color": "..." },
  "tasks": [{ "id": "...", "title": "...", "description": "...", "status": "...", "priority": "..." }]
}
```

### Hardware Projects
Location: `~/phoenix-sync/PROJECTS/*/README.md`
Scanned by `/api/projects` endpoint

### Infrastructure Config
Location: `~/.phoenix-infrastructure.yaml`
Contains all network IPs, device info, service URLs

## COMMON TASKS FOR CLAUDE

### Add a new nav section
1. Edit `public/index.html` line ~210 - add `<button data-section="newname">Name</button>`
2. Add `<section id="newname" class="section">` in HTML
3. Add `loadNewname()` function in JavaScript
4. Add to `loadSection()` switch statement

### Add a new API endpoint
1. Edit `src/server.js`
2. Add `app.get('/api/yourpath', async (req, res) => { ... })`
3. Restart server: `pkill -f "life-of-josh"; cd ~/life-of-josh && node src/server.js &`

### Add a new project type to /api/projects
Edit `src/server.js` line ~907 (hardware projects section)

## RESTART COMMAND
```bash
pkill -f "node.*server" 2>/dev/null; cd ~/life-of-josh && nohup node src/server.js > ~/loj.log 2>&1 &
```

## RELATED FILES
- Infrastructure: `~/.phoenix-infrastructure.yaml`
- Todos: `~/phoenix-sync/todo-system/categories/`
- Hardware Projects: `~/phoenix-sync/PROJECTS/`
- Phoenix Ecosystem: `~/repos/phoenix-forge-ecosystem/`

---
> Source: [dragonbornjedi-code/life-of-josh](https://github.com/dragonbornjedi-code/life-of-josh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
