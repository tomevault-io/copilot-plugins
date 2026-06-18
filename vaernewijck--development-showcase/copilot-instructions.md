## development-showcase

> Multi-display video showcase for Devine student projects.

# Development Showcase

Multi-display video showcase for Devine student projects.

## Quick Start

```bash
npm install
npm start
```

Open http://localhost:3000/setup.html

## Architecture

```
Server (server.js)
    ├── HTTP server (static files)
    └── WebSocket server (real-time sync)
         ├── Controller (role: 'controller')
         └── Displays (role: 'display', offset: 0/1/2)
```

## Key Files

- `server.js` - Node.js server with WebSocket
- `public/js/controller.js` - Controller logic
- `public/js/display.js` - Display logic
- `data/projects.json` - Assignment data (gitignored)
- `data/videos/` - Video files organized by assignment (gitignored)

## Display Sync Logic

When an assignment is selected:
1. Server calculates unique studentIndex for each display: `(offset + visitCount) % numStudents`
2. Each display receives its specific student
3. Revisiting same assignment shows different students (visitCount increments)
4. Extra displays (offset >= numStudents) show placeholder

## Autoplay Flow

1. Master display (offset=0, master=true) video ends
2. Master sends `videoEnded` to server
3. Server forwards to controller
4. Controller advances to next assignment (if autoplay enabled)

## WebSocket Messages

**Controller → Server:**
- `{ type: 'register', role: 'controller' }`
- `{ type: 'navigate', direction: 'goto', assignmentIndex, assignment }`

**Display → Server:**
- `{ type: 'register', role: 'display', offset: number }`
- `{ type: 'videoEnded' }` (master only)

**Server → Display:**
- `{ type: 'state', currentProject, studentIndex, assignment }`

**Server → Controller:**
- `{ type: 'state', currentProject }`
- `{ type: 'videoEnded' }`

---
> Source: [vaernewijck/development-showcase](https://github.com/vaernewijck/development-showcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
