## digitalkanbanplatform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-driven data visualization dashboard platform. Users describe requirements in natural language, and AI generates complete dashboards. Supports manual drag-and-drop editing, real-time data binding, and one-click HTML export.

**Core Philosophy**: "Connect data, say a sentence, export and deploy directly"

## Development Commands

### Frontend (Vue 3 + Vite)
```bash
cd frontend
npm install
npm run dev      # → http://localhost:5175
npm run build    # Production build
npm run preview  # Preview production build
```

### Backend (FastAPI + Python)
```bash
cd backend
pip install -r requirements.txt

# Configure AI (optional - bulk operations/theme switching work without it)
echo "MOONSHOT_API_KEY=your_key" > .env

python -m uvicorn app.main:app --port 8000 --reload
```

### Database Initialization (for testing)
```bash
# SQLite sample data (built-in)
cd backend && python init_sample_db.py

# MySQL test data (optional)
mysql -u root -p < doc/test_mysql_init.sql
```

## Architecture

### System Theme vs Board Style Separation

**Critical Design**: The platform separates **system theme** (UI shell) from **board style** (canvas visuals):

- **System Theme** (`applySystemTheme`): Controls app shell (topbar, sidebar, buttons)
- **Board Style** (`applyBoardStyle`): Controls canvas area only (background, component borders, chart colors)
- **AI Boundary**: AI's `CHANGE_THEME` command only affects board style, never system theme

Implementation: `frontend/src/stores/theme.js` (lines 23-62)

### Three-Layer Intent Routing

AI requests flow through three levels to minimize token usage:

```
User Message → Level 1: Regex (0 tokens, exact commands like "删除")
            → Level 2: Keywords (0 tokens, themes/colors/charts)
            → Level 3: LLM Agent (Kimi, scene generation/component modification)
```

**Design Principle**: "Better to miss and send to LLM than to misinterpret and do the wrong thing"

Configuration: `backend/app/ai_config.py`
Router: `backend/app/routers/ai.py`

### Command System

All operations (AI and manual) produce Command objects that are:
- Serializable JSON
- Undoable/redoable
- Uniform interface for both AI and user actions

Command types:
- `ADD_WIDGET`, `UPDATE_WIDGET`, `DELETE_WIDGET`
- `MOVE_WIDGET`, `RESIZE_WIDGET`
- `BATCH` (for multi-component operations)

Implementation: `frontend/src/core/command.js`

### Registry-Driven Architecture

All extensible components use a central registry pattern:

```javascript
// frontend/src/core/registry.js
registerWidget(type, config)   // 15 visualization components
registerTheme(name, config)     // 5 built-in + unlimited custom themes
registerDatasource(type, config) // API, Database adapters
```

To add new components/themes: register in corresponding `_registry.js` files, no core code changes needed.

### Data Model

```javascript
Project {
  id, name, theme,
  dataSources: []  // Project-level, shared across all dashboards
}

Dashboard {
  id, projectId, name,
  widgets: [],     // Each dashboard has its own component list
  createdAt, savedAt
}

Widget {
  id, type,
  props: { title, value, color, ... },  // All properties AI can modify
  position: { x, y },                    // AI can modify ("move to top-right")
  size: { w, h },                        // AI can modify ("make it bigger")
  dataSource: {                          // Component-level data binding (optional)
    sourceId: "ds_1",                    // References project-level datasource
    mapping: {                           // Field mapping
      x: "date",                         // X-axis field
      y: "amount",                       // Y-axis field
      series: "category",                // Series field (optional)
      value: "total"                     // KPI value field
    },
    interval: 30000                      // Auto-refresh interval (ms), 0=disabled
  }
}
```

### Hybrid Workflow

Users can freely mix AI generation and manual editing:

1. Create dashboard → Enter editor
2. Freely do any of these (no fixed order, repeatable):
   - Manually drag components from panel
   - Tell AI "add a KPI card"
   - Go to datasource page and configure an API
   - Return and tell AI "use that API data to make a line chart"
   - Manually adjust position and style
   - Tell AI "pick a few more suitable charts from the data"
   - Continue manual fine-tuning → Export

**Key Principle**: Canvas is always the source of truth. AI and manual operations both **add** to the canvas, never overwrite each other.

### AI Context

Every AI request must include full context:

```javascript
context = {
  widgets: [{ id, type, props, position, size, dataSource }],  // All components on canvas
  selectedId: "widget_xxx" | null,                             // Currently selected component
  dataSources: [{ id, name, type, fields, sample }],           // All available datasources
  theme: "dark-tech"                                           // Current theme
}
```

This enables AI to make informed decisions like:
- "add a chart" → AI sees sales API available, auto-selects line chart and binds data
- "change color" → AI sees a KPI card is selected, modifies its color property
- "add a few more" → AI sees 4 KPIs but no charts, adds line and pie charts

## Key Files

### Frontend Core
- `src/core/registry.js` - Central registry for widgets/themes/datasources
- `src/core/command.js` - Command system (execute/undo/redo)
- `src/core/event-bus.js` - Event bus using mitt

### Frontend Stores (Pinia)
- `src/stores/project.js` - Project list + current project + datasources
- `src/stores/dashboard.js` - Dashboard list + current dashboard + widgets
- `src/stores/theme.js` - System theme + board style separation
- `src/stores/history.js` - Undo/redo stack

### Frontend Components
- `src/components/Workspace.vue` - Canvas area (drag/resize/select)
- `src/components/PropEditor.vue` - Property panel (Style/Data tabs)
- `src/components/AiChat.vue` - AI conversation interface
- `src/widgets/` - 15 visualization components + WidgetWrapper

### Backend Core
- `app/routers/ai.py` - Three-layer intent routing + Agent dispatch
- `app/ai_config.py` - All AI constants (keywords, regex, prompts)
- `app/agents/scene.py` - Scene Agent (data-aware dashboard generation)
- `app/agents/component.py` - Component Agent (data-aware modifications)
- `app/services/kimi.py` - Kimi API wrapper (OpenAI SDK compatible)
- `app/services/api_probe.py` - API field detection (supports relative paths)
- `app/services/db_connector.py` - Database connection (SQLite/MySQL/PostgreSQL)

## Important Design Decisions

1. **App Shell Architecture**: Topbar + sidebar + router-view, editor is independent fullscreen route
2. **Project → Dashboard 1:N**: One project contains multiple dashboards, each manages components independently
3. **Command Pattern**: AI and manual operations output same JSON commands
4. **Hybrid Workflow**: AI generation and manual editing are not mutually exclusive, can alternate
5. **Three-Layer Routing Saves Tokens**: Simple operations handled locally via regex/keywords, complex needs go to LLM
6. **Persistence Adapter**: Unified save/load interface, currently localStorage, can swap to API later
7. **Schema-Driven Property Editing**: Component registry defines schema, PropEditor auto-generates forms
8. **Component-Level Data Binding**: Each widget can independently bind datasource + field mapping, runtime fetched by dataFetcher
9. **AI CHANGE_THEME Only Affects Board**: System theme and board style are decoupled, AI operations only affect board
10. **Mock API Self-Contained**: Built-in mock data endpoints, no external network dependency for full demo

## Testing

Comprehensive test manual: `doc/TEST_MANUAL.md` (75 test cases across 12 modules)

Key test areas:
- Project management
- Data source management (Mock, SQLite, MySQL, API)
- Dashboard editor operations
- Data binding with AI SQL inference
- AI-generated dashboards
- Smart data querying
- Theme switching (verify system theme vs board style separation)
- Export functionality

## Documentation

- `doc/STATUS.md` - Project status overview
- `doc/architecture_v4.md` - Frontend architecture specification (final version)
- `doc/TEST_MANUAL.md` - Complete testing manual
- `doc/implementation_plan.md` - Implementation roadmap
- `README.md` - Quick start guide

## Code Style Notes

- Frontend uses Vue 3 Composition API with `<script setup>`
- Backend uses FastAPI with async/await
- All AI configuration centralized in `backend/app/ai_config.py` (never hardcode keywords/prompts in routers)
- Component registration uses registry pattern (add to `_registry.js`, not core code)
- Persistence uses adapter pattern (currently localStorage, designed for future API swap)

---
> Source: [wzwwhh/digitalKanbanPlatform](https://github.com/wzwwhh/digitalKanbanPlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
