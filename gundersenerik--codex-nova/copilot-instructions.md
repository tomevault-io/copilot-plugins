## codex-nova

> This is the "Naming Standards Hub - Frontend Edition" - a **frontend-only** version that runs entirely in the browser using SQLite (via SQL.js) for data storage. Perfect for static hosting and offline usage. All code files are at the workspace root level.

# CJC - Codex Naming Standards Hub - AI Agent Instructions

## Quick Start

This is the "Naming Standards Hub - Frontend Edition" - a **frontend-only** version that runs entirely in the browser using SQLite (via SQL.js) for data storage. Perfect for static hosting and offline usage. All code files are at the workspace root level.

### Development Servers

- **Frontend (HTML)**: http://localhost:4200 - **USE THIS FOR DEVELOPMENT**

View changes at http://localhost:4200.

### How to Start/Stop Services

```bash
# Start both services
./start-dev.sh start

# Check status
./start-dev.sh status

# Stop all services
./start-dev.sh stop

# Restart services
./start-dev.sh restart
```


## ⚠️ IMPORTANT: Where to Make Changes

**ALWAYS use the Python dev server (port 4200) for changes!**

- ✅ **Correct**: http://localhost:4200 (it requires reloading)

## 📁 Project Structure

```
/home/coder/ (workspace root)
├── index.html                  # Main application page
├── css/
│   ├── main.css               # Core styles
│   ├── components.css         # Component styles  
│   └── responsive.css         # Mobile responsive styles
├── js/
│   ├── lib/
│   │   ├── sql-wasm.js       # SQL.js library (50KB)
│   │   └── sql-wasm.wasm     # SQLite WASM engine (599KB)
│   ├── config.js             # Static configuration
│   ├── sql-database.js       # SQLite database wrapper
│   ├── data-manager.js       # Data persistence & backups
│   ├── database.js           # Database service layer
│   ├── auth.js               # Simplified authentication
│   ├── promocode.js          # Promocode business logic
│   ├── navigation.js         # UI navigation logic
│   └── main.js               # Application initialization
├── database/
│   ├── schema.sql            # SQLite database schema
│   └── sample-data.sql       # Sample data for development
├── assets/
│   └── icons/                # Application icons
├── start-dev.sh              # Development server script
├── CLAUDE.md                 # This file
└── README.md                 # Project documentation
```

## 🚀 Common Tasks

### Start Development
```bash
./start-dev.sh start
```

### Commit code

After every change, commit the changes to git using `git commit -a -m {MESSAGE}`. Make sure that `{MESSAGE}` is a commit message that will make it easier for technical people to review later.

## 🔧 Troubleshooting

### View Not Changing?
- ✅ Check you're viewing http://localhost:4200 (not 8000)
- ✅ Try restarting the dev server: `./start-dev.sh restart`

### Services Not Starting?
```bash
# Check what's running
tmux list-sessions

# Stop everything and restart
./start-dev.sh stop
./start-dev.sh start
```

---

**Remember**: Always use http://localhost:4200 for development and style changes!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gundersenerik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
