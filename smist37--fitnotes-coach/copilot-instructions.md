## fitnotes-coach

> This project follows a modular structure. Keep code organized by purpose:

# CoachPWA Development Guidelines

## Code Organization Principles

This project follows a modular structure. Keep code organized by purpose:

| Directory | Purpose | Examples |
|-----------|---------|----------|
| `src/` | Server-side JavaScript | HTTP handlers, business logic, utilities |
| `public/` | Static frontend files | HTML, client JS, CSS, manifest |
| `context/` | Coach personality & memory | Persona, goals, commitments (markdown) |
| `scripts/` | Shell scripts & CLI tools | Sync scripts, startup, binaries |
| `config/` | Configuration files | MCP config, other JSON configs |
| `data/` | Runtime data (gitignored) | Databases, CSVs, logs, keys |

## Modularization Guidelines

### When to create a new module in `src/`

- **Extract when a file exceeds ~300 lines** or has multiple distinct responsibilities
- **Group related functions** into modules (e.g., `sync.js`, `push.js`, `routes.js`)
- **Use clear naming**: `<feature>.js` for feature modules, `<noun>-utils.js` for utilities

### Suggested `src/` structure for future refactoring

```
src/
├── server.js           # Entry point, HTTP server setup, route registration
├── fitnotes-mcp.js     # MCP server (tools for Claude)
├── routes/             # HTTP route handlers (if server.js grows)
│   ├── chat.js
│   ├── sync.js
│   └── push.js
├── services/           # Business logic
│   ├── claude.js       # Claude CLI integration
│   ├── fitnotes.js     # FitNotes database queries
│   └── cronometer.js   # Cronometer data parsing
└── utils/              # Shared utilities
    ├── logger.js
    └── paths.js        # PROJECT_ROOT and path constants
```

### Module patterns

```javascript
// Export functions, not classes (keep it simple)
// src/utils/logger.js
const fs = require('fs');
const path = require('path');

const LOG_FILE = path.join(__dirname, '../../data/fitnotes.log');

function log(level, message) {
  const timestamp = new Date().toISOString();
  fs.appendFileSync(LOG_FILE, `[${timestamp}] [${level}] ${message}\n`);
}

module.exports = { log };
```

```javascript
// Import in other files
const { log } = require('./utils/logger');
```

### Frontend (`public/`)

- Keep `index.html` as a single-file PWA for simplicity
- If JS grows complex, extract to `public/js/app.js`
- Service worker (`sw.js`) should remain minimal

## Project Structure

```
/opt/projects/fitnotes-coach/
├── package.json            # npm config
├── .env                    # Environment variables
│
├── public/                 # Static frontend files
│   ├── index.html          # PWA entry point
│   ├── index.v2.html       # Alternative frontend
│   ├── sw.js               # Service worker
│   └── manifest.json       # PWA manifest
│
├── src/                    # Server-side source code
│   ├── server.js           # Node.js backend (entry point)
│   └── fitnotes-mcp.js     # MCP server for tools
│
├── context/                # Coach context files (read by Claude)
│   ├── coach-persona.md    # System prompt / personality
│   ├── client-profile.md   # Client info
│   ├── goals.md            # Fitness goals
│   ├── commitments.md      # Accountability tracking
│   ├── excuses.md          # Excuse history
│   └── recipes.md          # Custom recipes
│
├── scripts/                # Shell scripts & CLI tools
│   ├── start.sh            # Startup script
│   ├── cronometer-sync.sh  # Nutrition sync
│   └── cronometer-export   # Binary (gitignored)
│
├── data/                   # Runtime data (gitignored)
│   ├── *.fitnotes          # FitNotes backups
│   ├── cronometer-*.csv    # Nutrition data
│   └── fitnotes.log        # Log file
│
└── config/                 # Configuration
    └── mcp-config.json     # MCP server config
```

## After Adding Features

1. **Restart the server** to apply changes:
   ```bash
   # Production (normal usage)
   sudo systemctl restart fitnotes-coach

   # Testing only - ok to use node directly
   pkill -f "node src/server.js" ; node src/server.js
   ```

2. **Commit changes to git**:
   ```bash
   git add -A && git commit -m "Description of feature" && git push
   ```

---
> Source: [smist37/fitnotes-coach](https://github.com/smist37/fitnotes-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
