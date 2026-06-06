## project-mgmt-app

> > **IMPORTANT**: Keep this file updated when making significant changes to the codebase. This file serves as persistent memory between Claude Code sessions.

# Project Management App - Project Notes

> **IMPORTANT**: Keep this file updated when making significant changes to the codebase. This file serves as persistent memory between Claude Code sessions.

## Overview
Project management module with projects, tasks, task relations, inline editing, and modern glassmorphism UI.

## Data Model

### Task
- Fields: name, number, status, type, priority, description, dueDate, assignee, estimatedHours, completionPercentage, tags
- `@ManyToOne Project project` - every task belongs to a project
- `parentRelationships` (mappedBy="parent", cascade PERSIST+MERGE) = relations where THIS task is the PARENT
- `childRelationships` (mappedBy="child", cascade PERSIST+MERGE) = relations where THIS task is the CHILD
- `getRelationships()` returns union of both sets
- Number auto-generated in `TaskService.save()` as `{projectNumber}-{random4-5digits}`

### TaskRelation
- `@Id UUID id` - **NO @GeneratedValue** - MUST set `relation.setId(UUID.randomUUID())` before persist
- `Task parent`, `Task child` - directional relationship
- `TaskRelationshipType type` - enum stored as STRING
- No dedicated repository - persisted via CASCADE on Task's relationship collections

### TaskRelationshipType (enum)
Each value has `displayName` (parent perspective) and `inverseDisplayName` (child perspective):
| Enum | displayName | inverseDisplayName |
|------|-------------|-------------------|
| `CHILD_IS_PART_OF` | "Parent of" | "Is part of" |
| `BLOCKS` | "Blocks" | "Is blocked by" |
| `REQUIRES` | "Requires" | "Is required by" |
| `RELATES_TO` | "Relates to" | "Relates to" |
| `CHILD_SOLVES` | "Solved by" | "Solves" |

### Project
- Fields: name, number, status, priority, description
- `@OneToMany Set<Task> tasks`

## Views

### AbstractTaskDetailsView
- Extends `AbstractPageView`, implements `HasUrlParameter<String>`
- **CRITICAL**: `init()` must clear old children first: `getChildren().filter(c -> c != projectsPageLayout).toList().forEach(this::remove)` - view instance is reused on navigation
- Sections: Header (type icon + number + name + inline-edit status/priority), Metadata grid (inline-edit fields), Description (WYSIWYG), Relations (TaskRelationsPanel)

### AbstractProjectDetailsView
- Same pattern as TaskDetailsView - must clear children in `init()`
- Sections: Header, Stats row (total/open/in-progress/done/overdue), Description, Task list (ProjectTaskListView)

### TaskRelationsPanel
- Replaces old SubtaskListPanel
- Groups all relations by type+direction (e.g., "Blocks (2)", "Is blocked by (1)")
- Each group has collapsible header with count badge
- Rows: type icon + number + name + status badge + remove button (×)
- Add row: ComboBox (relation type) + ComboBox (task search with lazy loading) + Link button + New button
- Inverse relations: when user selects "Is blocked by", creates relation with parent=target, child=current (swaps direction)
- `refreshRelations()` reloads task from DB to get fresh relationships

### ProjectTaskListView
- Extends `AbstractBervanTableView` for task list within a project
- `customizePreLoad()` filters by project.id
- Reuses `AbstractAllTasksListView.customizeTaskColumns()` for badge renderers

## Key Components

### InlineEditableField
- Click-to-edit component with FieldType enum: TEXT, COMBOBOX, DATE, NUMBER
- Display mode: Span with value + pencil icon on hover
- Edit mode: appropriate input component
- Save: Enter/blur, Cancel: Escape
- `Consumer<Object> onSave` callback

### TaskTypeIconHelper
- Maps type string → icon + color
- Task → CHECK_SQUARE_O (info), Bug → BUG (danger), Story → BOOK (success), Feature → LIGHTBULB (accent), Objective → BULLSEYE (warning)
- `createIcon()` and `createLargeIcon()` methods

### StatusBadgeHelper
- `createStatusBadge(status)` - colored pill Span with CSS class `.task-badge.status-*`
- `createPriorityBadge(priority)` - colored pill Span with CSS class `.task-badge.priority-*`
- `createMiniProgress(percentage)` - tiny progress bar Div

## Common Pitfalls

1. **TaskRelation ID**: No @GeneratedValue - always set `relation.setId(UUID.randomUUID())` before persist
2. **View instance reuse**: HasUrlParameter views are reused - clear old components in `init()`
3. **ComboBox data provider**: Must call `query.getLimit()` and `query.getOffset()` - otherwise IllegalStateException
4. **Relation direction**: When displaying, check both parentRelationships (use displayName) AND childRelationships (use inverseDisplayName)
5. **Cascade persistence**: Relations are persisted by adding to parent task's collection and saving the task - no direct repository

## CSS (bervan-project-mgmt.css)
Located in `common/.../frontend/bervan-project-mgmt.css`. Key classes:
- `.pm-section` - Section panel (full width, box-sizing, padding, border-radius)
- `.inline-editable-field` - Click-to-edit styling with hover pencil
- `.task-badge.status-*` / `.priority-*` - Colored pill badges
- `.relation-group`, `.relation-group-header`, `.relation-row` - Grouped relation display
- `.relation-remove-btn` - Remove button (visible on row hover)
- `.task-detail-header` - Flex header for task details
- `.task-metadata-grid` - CSS grid 2→1 columns (responsive)
- `.tag-chip` - Tag pill with remove button
- `.project-stats-row` / `.project-stat-card` - Project statistics cards

## Config
- `src/main/resources/autoconfig/Task.yml` - Column config (form/table visibility, validation)
- `src/main/resources/autoconfig/Project.yml` - Project column config

---
> Source: [vnhk/project-mgmt-app](https://github.com/vnhk/project-mgmt-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
