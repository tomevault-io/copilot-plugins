## project-tracker

> Guidance for Claude Code when working with this repository.

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## What This Is

Project Tracker is a local web tool for organizing projects. It has:

- **Frontend**: HTML files (`project-tracker.html`, `claude-tracker.html`, `git-tracker.html`) with shared CSS/JS in `shared/`
- **Backend**: TypeScript Express server in `server/src/`, compiled to `server/dist/`

## Commands

```bash
# Install and run
cd server && npm install && npm start
# Server runs on http://localhost:3001

# Development mode (auto-reloads on changes)
cd server && npm run dev

# Build TypeScript only
cd server && npm run build

# Run tests
cd server && npm test

# Open frontend
open project-tracker.html  # or just double-click it
```

## Architecture

### Backend (`server/src/`)

TypeScript source compiled to `server/dist/`. Entry point: `src/index.ts` wires up services and starts the server. `src/app.ts` creates the Express app (separated for testability).

Instance-based service classes (constructor-injected dependencies):
- `CacheManager` (`services/cache-manager.ts`) - File-based caching with 1-hour TTL
- `ProjectAnalyzer` (`services/project-analyzer.ts`) - Scans directories, detects tech stacks, generates tags
- `ClaudeAnalyzer` (`services/claude-analyzer.ts`) - Detects CLAUDE.md files and .claude directories
- `GitAnalyzer` (`services/git-analyzer.ts`) - Git status, branch info, GitHub detection
- `UserData` (`services/user-data.ts`) - Persists tags and favorites

Route handlers are in `src/routes/` (one file per resource: projects, directories, tags, favorites, git, claude, health).

Tag generation uses data-driven rules from `src/config/tag-rules.json` (editable without code changes).

Types are defined in `src/types.ts`.

Data files (in `server/`, not `src/`):
- `directories.json` - Configured scan directories
- `user-data.json` - Tags and favorites (persistent)
- `projects-cache.json` - Cached scan results

### Frontend

HTML files link shared assets from `shared/`:
- `shared/shared.css` - Common styles (reset, nav, modals, theme, responsive)
- `shared/shared.js` - Common JS (escapeHtml, theme toggle, notifications, directory/tag management)

Each HTML file has only page-specific CSS and JS. All dynamic data is sanitized via `escapeHtml()` / `escapeJsStr()` before `innerHTML` injection.

Communicates with backend at `http://localhost:3001/api`. Falls back to hardcoded data if backend is unavailable.

## API Endpoints

### Projects
- `GET /api/projects` - Full scan
- `GET /api/projects/cached` - Return cache, trigger background scan if stale

### Configuration
- `GET /api/config` - Current directories
- `POST /api/directories` - Add directory (validates path exists)
- `DELETE /api/directories` - Remove directory
- `PUT /api/directories` - Replace all directories

### User Data
- `GET /api/favorites` - List favorites
- `POST /api/favorites` - Add favorite `{ projectPath }`
- `DELETE /api/favorites` - Remove favorite `{ projectPath }`
- `GET /api/projects/:path/tags` - Get tags for project
- `POST /api/projects/:path/tags` - Save tags `{ tags: [] }`

### Other
- `GET /api/health` - Health check
- `GET /api/tags` - All tags across projects
- `DELETE /api/tags/:tagName` - Remove a tag from all projects
- `GET /api/claude/projects` - Projects with CLAUDE.md
- `GET /api/git/repos` - Git repository details

## Key Behaviors

- Projects are scanned from directories listed in `directories.json`
- Each directory should be a project root, not a parent folder
- WSL path conversion: Windows paths like `C:\...` become `/mnt/c/...`
- Cache expires after 1 hour
- Tags merge auto-detected tags with user-saved tags
- Favorites and tags persist in `user-data.json`
- Tag rules in `src/config/tag-rules.json` drive project-specific tag detection

## Adding Features

### New detection in ProjectAnalyzer
1. Add to `detectTechnologies()` in `src/services/project-analyzer.ts` for file extensions
2. Add to `detectCategory()` for project types
3. Add to `generateTags()` for auto-tagging, or add a rule to `src/config/tag-rules.json`

### New API endpoint
1. Create a route file in `src/routes/`
2. Register it in `src/app.ts`
3. Add types to `src/types.ts` if needed

### Frontend changes
- Shared styles/logic: edit `shared/shared.css` or `shared/shared.js`
- Page-specific: edit the relevant HTML file's `<style>` or `<script>` block

## Testing

Tests use Jest with ts-jest. Test files are in `server/src/__tests__/`.

- **Unit tests** (`__tests__/services/`): CacheManager, ProjectAnalyzer, ClaudeAnalyzer, GitAnalyzer, UserData
- **API tests** (`__tests__/routes/`): health, favorites, tags, directories (using supertest)
- **Test helpers** (`__tests__/test-helpers.ts`): factories for creating isolated test instances

Run `npm test` from `server/`.

## Dependencies

Runtime: `express`, `cors`
Dev: `typescript`, `tsx`, `jest`, `ts-jest`, `supertest` + type definitions

No database -- everything is JSON files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bshepp-projects)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bshepp-projects)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
