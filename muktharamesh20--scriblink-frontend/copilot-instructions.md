## scriblink-frontend

> ScribLink Vue.js project structure and architecture guide


# ScribLink Project Structure

## Core Architecture
- **Framework**: Vue 3 with Composition API
- **Build Tool**: Vite
- **State Management**: Pinia
- **Routing**: Vue Router
- **HTTP Client**: Axios
- **Markdown**: marked + highlight.js

## File Organization
- **Entry Point**: [main.js](mdc:src/main.js) - Vue app initialization
- **Root Component**: [App.vue](mdc:src/App.vue) - Main app layout, theme management, auth state
- **Global Styles**: [style.css](mdc:src/style.css) - CSS variables, theme system, global styles
- **Components**: [src/components/](mdc:src/components/) - All Vue components
- **Services**: [src/services/](mdc:src/services/) - API calls, authentication, business logic

## Key Components
- **Dashboard**: [Dashboard.vue](mdc:src/components/Dashboard.vue) - Main dashboard with folders, notes, tags
- **NoteEditor**: [NoteEditor.vue](mdc:src/components/NoteEditor.vue) - Markdown editor with preview/edit modes
- **FolderTree**: [FolderTree.vue](mdc:src/components/FolderTree.vue) - Folder navigation sidebar
- **SummaryPanel**: [SummaryPanel.vue](mdc:src/components/SummaryPanel.vue) - Note summary display/edit
- **TagsPanel**: [TagsPanel.vue](mdc:src/components/TagsPanel.vue) - Note tags management

## Services Layer
- **API Services**: [apiServices.js](mdc:src/services/apiServices.js) - Unified API interface
- **Auth Service**: [authService.js](mdc:src/services/authService.js) - Authentication management
- **API Client**: [api.js](mdc:src/services/api.js) - Axios configuration and base API calls

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muktharamesh20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
