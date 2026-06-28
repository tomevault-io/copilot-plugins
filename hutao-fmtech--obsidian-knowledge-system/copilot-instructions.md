## obsidian-knowledge-system

> This directory is an **Obsidian Vault** implementing a comprehensive Personal Knowledge Management (PKM) system. It combines **PARA**, **Zettelkasten**, and **MOC (Map of Content)** methodologies to organize information, manage projects, and foster creativity.

# Life Knowledge System - Gemini Context

This directory is an **Obsidian Vault** implementing a comprehensive Personal Knowledge Management (PKM) system. It combines **PARA**, **Zettelkasten**, and **MOC (Map of Content)** methodologies to organize information, manage projects, and foster creativity.

## 📂 System Architecture

The vault follows a strict numeric folder structure to organize different types of content:

| Directory | Name | Methodology | Purpose |
| :--- | :--- | :--- | :--- |
| `00-系统/` | System | - | Configuration, Templates (`templates/`), Dashboard (`🏠 HOME.md`), and Manuals. |
| `01-收件箱/` | Inbox | GTD | Temporary capture zone for quick notes and ideas. Must be cleared daily. |
| `02-项目/` | Projects | PARA | Active projects with deadlines (e.g., "Weight Loss Plan"). |
| `03-领域/` | Areas | PARA | Ongoing responsibilities without deadlines (e.g., "Health", "Finance"). |
| `04-资源/` | Resources | PARA | Reference materials, reading notes, and learning resources. |
| `05-归档/` | Archives | PARA | Completed projects and inactive items, organized by year. |
| `10-卡片盒/` | Zettelkasten | Zettelkasten | Atomic, permanent notes (Concept, Method, Index/MOC, Opinion). |
| `20-日志/` | Journals | Periodic | Temporal records: Daily, Weekly, Monthly, Yearly logs. |
| `30-输出/` | Output | - | Content creation: Blog posts, speeches, shared articles. |
| `90-附件/` | Attachments | - | Static assets: Images, PDFs, Audio files. |

## 🔑 Key Files & Entry Points

*   **`00-系统/🏠 HOME.md`**: The central dashboard. Use this to orient yourself. It contains Dataview queries for active projects, tasks, and recent notes.
*   **`00-系统/📖 使用手册.md`**: The official User Manual. Refer to this for specific workflow rules.
*   **`00-系统/🗺️ 知识地图.canvas`**: A visual navigation map of the system.
*   **`CLAUDE.md`**: Contains project-specific rules and high-level architectural guidelines.
*   **`00-系统/templates/`**: Contains `.md` templates for various note types. **Always use these when creating new files.**

## ⚙️ Technical Stack & Plugins

This system relies heavily on specific Obsidian plugins. When analyzing files, be aware of their syntax:

*   **Dataview**: Used extensively for dynamic lists and tables. Look for code blocks starting with ````dataview` or ````dataviewjs`.
    *   *Usage:* Querying metadata (YAML frontmatter) to generate dashboards.
*   **Templater**: Used for file creation automation.
*   **Calendar / Periodic Notes**: Manages the `20-日志` folder structure.
*   **Obsidian Tasks**: For task management within markdown files.
*   **Excalidraw**: For whiteboard/diagram files (`.excalidraw.md`).

## 🔄 Core Workflows

### 1. Capture & Process
*   **Inbox (`01-收件箱/`)**: New ideas land here first.
*   **Processing Rule**: Move items to their permanent home (`02` to `10`) based on their nature.
    *   Actionable & Deadline? -> **Project** (`02`)
    *   Ongoing Responsibility? -> **Area** (`03`)
    *   Reference Material? -> **Resource** (`04`)
    *   Atomic Knowledge? -> **Zettelkasten** (`10`)

### 2. Knowledge Creation (Zettelkasten)
*   **Atomic Notes**: Each note in `10-卡片盒` should represent *one* concept.
*   **Linking**: Notes must be heavily linked using `[[WikiLinks]]`. Avoid "orphan" notes.
*   **Types**:
    *   **Concept**: Definitions (e.g., "Entropy").
    *   **Method**: How-to guides (e.g., "How to use Git").
    *   **Opinion**: Personal thoughts/essays.
    *   **Index (MOC)**: Maps of Content that aggregate links to other notes.

### 3. Review Cycles
*   **Daily**: Journaling in `20-日志/日记/`.
*   **Weekly**: Review Projects (`02`) and update Area (`03`) health.
*   **Monthly**: High-level replanning and archiving.

## 📝 Conventions

*   **File Naming**:
    *   Daily Notes: `YYYY-MM-DD.md`
    *   Weekly Notes: `YYYY-Wxx.md`
    *   Knowledge Notes: Descriptive, concise titles (e.g., `Zettelkasten Method.md`).
*   **Frontmatter (YAML)**: Every file should have a YAML header.
    *   Required: `created`, `tags`.
    *   Context-specific: `status` (Projects), `author` (Books), `type` (Cards).
*   **Tags**: Use hierarchical tags (e.g., `#Work/Project`, `#Learning/Reading`).

## 🤖 Agent Instructions (How to help)

1.  **Reading Context**: When asked to explain a concept or project status, check `02-项目`, `03-领域`, and `10-卡片盒` first.
2.  **Creating Content**:
    *   **Always** read the relevant template from `00-系统/templates/` before creating a file.
    *   **Always** add valid YAML frontmatter.
    *   **Always** use `[[Links]]` to connect new content to existing nodes.
3.  **Search Strategy**:
    *   Use `search_file_content` to find specific terms across the vault.
    *   Use `glob` to find files within specific PARA categories (e.g., `04-资源/**/*.md`).
4.  **Dataview**: You cannot "run" Dataview queries, but you can read the code blocks to understand *what* information is being aggregated and reproduce a similar search using your tools.

---
> Source: [hutao-fmtech/obsidian-knowledge-system](https://github.com/hutao-fmtech/obsidian-knowledge-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
