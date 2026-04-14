## datra

> Auto-generated from all feature plans. Last updated: 2026-01-27

# datra Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-01-27

## Active Technologies
- Go 1.23, React 18.2.0 + Wails v2.11.0, `gopkg.in/yaml.v3`, Lucide React (002-i18n-settings-page)
- `~/.datra/settings.yaml` (002-i18n-settings-page)
- React 18.2.0 (Frontend), Go 1.23 (Backend) + `react-router-dom`, `lucide-react` (004-frontend-routing)
- N/A (Frontend state only) (004-frontend-routing)
- React 18.2.0 (Frontend), Go 1.23 (Backend), TypeScrip + `@tanstack/react-router`, `lucide-react` (004-frontend-routing)
- React 18.2.0 (Frontend), TypeScript (Frontend) + `lucide-react` (Icons), `@tanstack/react-router` (Routing), `react-resizable-panels` (005-global-sidebar)
- N/A (Consumes existing `GetConnections` API) (005-global-sidebar)
- Go 1.23, React 18.2.0, TypeScript 5.x + Wails v2.11.0, `lucide-react`, `@tanstack/react-router`, `gopkg.in/yaml.v3`, `zalando/go-keyring` (Proposed for secure storage) (006-manage-connections)
- `~/.datra/connections.yaml` for metadata, OS Keychain/Credential Manager for passwords. (006-manage-connections)
- Go 1.23, React 18.2.0, TypeScript 5.x + Wails v2, @tanstack/react-query, @tanstack/react-router, @tanstack/react-table, @tanstack/react-virtual, zustand, lucide-reac (008-enhance-db-ui)
- N/A (Connects to external databases) (008-enhance-db-ui)

- Go 1.23 (Backend), React 18.2.0 (Frontend) + Wails v2.11.0, Lucide React (Icons) (001-empty-state-guide)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Go 1.23 (Backend), React 18.2.0 (Frontend)

## Code Style

Go 1.23 (Backend), React 18.2.0 (Frontend): Follow standard conventions

## Recent Changes
- 008-enhance-db-ui: Added Go 1.23, React 18.2.0, TypeScript 5.x + Wails v2, @tanstack/react-query, @tanstack/react-router, @tanstack/react-table, @tanstack/react-virtual, zustand, lucide-reac
- 007-connection-manager: Added [if applicable, e.g., PostgreSQL, CoreData, files or N/A]
- 006-manage-connections: Added Go 1.23, React 18.2.0, TypeScript 5.x + Wails v2.11.0, `lucide-react`, `@tanstack/react-router`, `gopkg.in/yaml.v3`, `zalando/go-keyring` (Proposed for secure storage)


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kayooliveira)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kayooliveira)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
