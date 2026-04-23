## tartb

> Provides the glass-morphism container with:

# TArtB - Art New Tab Chrome Extension

A Chrome extension that replaces your new tab page with stunning artwork from world-renowned museums.

## Tech Stack

- **Framework:** React 19 + TypeScript + Vite 7
- **State:** Zustand 5 (with Chrome storage persistence)
- **Data Fetching:** TanStack React Query 5
- **Backend:** Firebase Firestore + Cloud Functions
- **Styling:** Tailwind CSS 4 + Framer Motion
- **Date/Calendar:** date-fns + react-day-picker v9
- **Popover/Positioning:** @floating-ui/react
- **Drag & Drop:** @dnd-kit/core + @dnd-kit/sortable
- **i18n:** i18next (EN, VI)
- **Extension:** Manifest v3, new tab override

## Project Structure

```
src/
├── App.tsx                    # Main app, language sync, dock + sidebar wiring
├── main.tsx                   # React Query setup, i18n import
├── i18n.ts                    # i18next config
├── components/
│   ├── Artwork/
│   │   └── InteractiveArtwork.tsx  # Canvas-based image with dissolve effect
│   ├── icons/                 # Grip (drag handle icon)
│   ├── atoms/                 # GlassButton, Glass, Switch, Selector, Clock, Typography, WidgetWrapper, Popover, DatePicker, Dropdown, Toast
│   ├── molecules/             # ArtworkInfo, MenuCategories, DockStation, ToDo (with drag-drop components)
│   └── organisms/             # Sidebar (settings panel), DynamicFieldRenderer
├── assets/
│   └── icons/                 # SVG icons (Close, CheckCircle, Error, Info, Flag, Calendar, etc.)
├── stores/
│   ├── artworkStore.ts        # Current artwork state
│   ├── settingsStore.ts       # Widget + app settings with Chrome storage middleware
│   ├── todoStore.ts           # Task lists & tasks with Chrome storage persistence & error handling
│   └── toastStore.ts          # Toast notifications with auto-dismiss
├── services/
│   ├── api/
│   │   ├── artInstituteApi.ts # Chicago Art Institute API
│   │   ├── metMuseumApi.ts    # Metropolitan Museum API
│   │   └── wikiArtApi.ts      # WikiArt API (needs auth)
│   ├── firebase/
│   │   └── firestoreService.ts # Primary data source
│   └── todo/
│       └── todoService.ts      # Hybrid service: LocalTodoService + FirestoreTodoService with granular operations
├── hooks/
│   ├── useArtwork.ts          # Main hook: Firestore → API fallback chain
│   └── useToDo.ts             # ToDo state: input, priority, deadline, task CRUD, drag-drop reordering
├── types/
│   ├── settings.ts            # WidgetId enum, UserSettings, widget state interfaces
│   └── toDo.ts                # Task, TaskList, Tag, TaskGroup, TodoData interfaces
├── utils/
│   ├── stringUtils.ts         # generateId() - UUID generation with crypto API fallback
│   ├── objectUtils.ts         # removeUndefined() - clean objects before Firestore writes
│   ├── taskSortUtils.ts       # Task sorting logic (DATE, DUE_DATE, PRIORITY, TITLE, MANUAL)
│   ├── taskGroupUtils.ts      # Task grouping logic (PRIORITY, DATE, DUE_DATE, NONE)
│   └── taskReorderUtils.ts    # Drag-drop order calculation, cross-group property updates
├── constants/
│   ├── common.ts              # Enums: TimeFormat, ClockType, Language, FieldType, TaskPriorityType, TaskSortBy, TaskGroupBy
│   ├── widgets.ts             # WIDGET_REGISTRY - widget metadata (icons, names, categories)
│   ├── toDoConfig.ts          # PRIORITY_CONFIG - priority colors/labels
│   └── settingConfig.ts       # Settings UI structure by category
├── locales/                   # en.json, vi.json
└── animations/                # Framer Motion variants
```

## Key Patterns

### Widget System (macOS-style)
Each widget has two states:
- **enabled:** Widget appears in dock (red dot disables → removes from dock)
- **visible:** Widget appears on screen (yellow dot minimizes → stays in dock, dimmed)

```typescript
// Widget IDs
enum WidgetId {
  CLOCK = "clock",
  DATE = "date",
  ARTWORK_INFO = "artworkInfo",
  TODO = "toDo",
}

// Settings structure
interface UserSettings {
  widgets: {
    clock: { enabled: boolean, visible: boolean, type: ClockType, timeFormat: TimeFormat },
    date: { enabled: boolean, visible: boolean },
    artworkInfo: { enabled: boolean, visible: boolean },
  },
  artwork: { museum: string, changeInterval: number },
  app: { language: Language },
}
```

### DockStation Behavior
- Shows icons for **enabled** widgets only
- Dimmed icons for **minimized** widgets (enabled but not visible)
- **Click:** Toggle `visible` state
- **Long press (400ms):** Open settings for that widget's category

### WidgetWrapper
Provides the glass-morphism container with:
- 3D tilt effect on hover
- Yellow dot (minimize) and red dot (close) buttons
- Can connect directly to store via `widgetId` prop

### Data Fetching (useArtwork hook)
```
Firestore → Selected Museum API → Random Museum → Met Museum (fallback)
```
- React Query handles caching (based on changeInterval)
- Auto-refetch based on `settings.artwork.changeInterval`
- Progressive image loading (small → full)

### Todo Service Layer (Optimized)

**Architecture:** Hybrid service switches between LocalTodoService (anonymous users) and FirestoreTodoService (authenticated users)

**Service Interface:**
```typescript
interface TodoService {
  // Bulk operations (for initial load, migration, bulk deletes)
  load(), saveLists(), saveTasks(), saveTags(), clear()

  // Granular operations (for single-item CRUD)
  saveList(), deleteListById()
  saveTask(), updateTaskFields(), deleteTaskById()
  saveTag(), deleteTagById()

  // Bulk operations for specific scenarios
  deleteTasks(ids[])           // Bulk delete (clearCompleted)
  updateTasksOrders(updates[]) // Batch reorder (normalizeTaskOrders)
}
```

**LocalTodoService (Chrome Storage/localStorage):**
- In-memory cache (`TodoData | null`) populated on first `load()`
- Granular methods: modify cache → write full data to storage
- Cache ensures no redundant reads from storage
- ~50KB memory overhead (acceptable for Chrome extension)

**FirestoreTodoService (Cloud Firestore):**
- Granular methods use `setDoc()`, `updateDoc()`, `deleteDoc()` directly
- **No `getDocs()` calls** for single-item operations → massive savings
- Batch operations use `writeBatch()` for multi-item updates
- Real-time sync via `onSnapshot()` listeners on lists/tasks/tags collections

**Performance Impact:**
- Before: Adding 1 task = 100 reads + 101 writes = **201 Firestore operations**
- After: Adding 1 task = 0 reads + 1 write = **1 Firestore operation**
- **99.5% reduction** in Firestore billable operations for single-item changes

**Migration Strategy:**
- Smart merge on first sign-in: items with same ID use newer `updatedAt`
- Local data merged into Firestore, then real-time sync takes over
- Batch methods kept for backwards compatibility

### Settings Store Actions
```typescript
toggleWidgetVisible(widgetId)  // Click on dock icon
minimizeWidget(widgetId)       // Yellow dot
closeWidget(widgetId)          // Red dot
setWidgetEnabled(widgetId, enabled)  // From settings panel
restoreWidget(widgetId)        // Make visible again
```

### Popover System
Shared popover component (`atoms/Popover.tsx`) using @floating-ui/react:
- **Popover**: Root component, manages open state + positioning
- **PopoverTrigger**: Wraps the element that opens the popover
- **PopoverContent**: Portal-rendered content with Glass styling
- Auto-positioning with `flip`, `shift`, `offset` middleware
- Click-outside and Escape key dismissal via `useDismiss`
- Supports controlled (`open`/`onOpenChange`) or uncontrolled state

Used by: `Dropdown`, `DatePicker`, and future popovers (tags, task detail, etc.)

### Toast System & Error Handling
Comprehensive error handling with user feedback via toast notifications:

**Toast Store** (`stores/toastStore.ts`):
- Auto-dismiss with configurable durations (success: 3s, error: 5s, info: 4s)
- Prevents duplicate toasts (same message + type)
- Limits to max 3 visible toasts (stack management)
- Support for action buttons (e.g., retry on errors)

**Toast Component** (`atoms/Toast.tsx`):
- Glass-morphism styled notifications
- Framer Motion animations (slide in/out from top-right)
- Three types with icons: success (✓), error (⚠), info (ℹ)
- Manual close button + auto-dismiss
- Z-index: 100 (above all other UI)

**Error Handling Pattern** (applied to all 15 async store actions):
```typescript
try {
  const previousState = current; // Snapshot for rollback
  setLoading(key, true);

  set({ newState }); // Optimistic update
  await todoService.save(); // Persist to backend
} catch (error) {
  set({ previousState }); // Rollback on failure
  console.error("[TodoStore] Error:", error);
  throw error; // Bubble to hook for toast
} finally {
  setLoading(key, false);
}
```

**Loading States** (`todoStore.loading`):
```typescript
interface TodoLoadingState {
  isAddingTask, isUpdatingTask, isTogglingTask, isReorderingTask,
  isDeletingTask, isAddingList, isUpdatingList, isDeletingList,
  isClearingCompleted
}
```

**Hook-Level Toast Notifications** (`useToDo.ts`):
- Success toasts for user-initiated actions (add task, toggle task, add list)
- Error toasts with retry buttons for failed operations
- No success toast for drag-drop (too noisy)
- All errors are caught and displayed to user (zero silent failures)

**Translation Keys** (`locales/*.json`):
- Success: `toDo.toast.taskAdded`, `taskCompleted`, `listAdded`, etc.
- Error: `toDo.toast.errorAddingTask`, `errorTogglingTask`, etc.
- Action: `toDo.toast.retry`

### Task Utilities (Phase 1: Extracted Logic)

Complex filtering, grouping, and reordering logic extracted from `useToDo` hook into pure, testable utility functions:

**taskSortUtils.ts** - Task sorting logic:
```typescript
sortTasks(tasks: Task[], sortBy: TaskSortBy): Task[]
```
- DATE: Sort by createdAt (newest first)
- DUE_DATE: Tasks with deadline first (sorted by deadline), then tasks without deadline, order as tiebreaker
- PRIORITY: High→Medium→Low→None, order as tiebreaker
- TITLE: Alphabetical by title
- MANUAL: Sort by order field only

**taskGroupUtils.ts** - Task grouping logic:
```typescript
groupTasks(tasks: Task[], groupBy: TaskGroupBy, t: TranslateFunction): TaskGroup[]
```
- PRIORITY: Groups by High/Medium/Low/None priority (droppable)
- DATE: Groups by Today/Tomorrow/Earlier/Later creation date (not droppable - immutable)
- DUE_DATE: Groups by Overdue/Upcoming/NoDate deadline status (NoDate droppable to clear deadline)
- NONE: Single group for all incomplete tasks
- Always appends Completed group if completed tasks exist (droppable to mark as completed)

**taskReorderUtils.ts** - Drag-drop calculations:
```typescript
calculateNewTaskOrder(targetTasks, targetIndex, getTaskOrder?): number
getPropertyUpdatesForCrossGroupDrag(sourceGroupId, targetGroupId, targetGroupValue): TaskPropertyUpdates
createNormalizedOrderGetter(filteredTasks): (task: Task) => number
```
- Fractional ordering: Insert between existing tasks without full reordering
- Cross-group drag updates: priority changes, deadline clearing, completion toggling
- Normalized order getter: Used when switching from DATE/TITLE to MANUAL sort

**Benefits:**
- ~150 lines removed from useToDo hook (500 → 350 lines)
- Pure functions → easily testable in isolation
- Reusable across different components
- Clear separation of concerns (UI logic vs business logic)

### ToDo Widget
- **ToDoForm** (`molecules/toDo/toDoForm.tsx`): Task input with priority dropdown, date picker, and tags
  - Loading overlay with spinner during task addition
  - Input and submit button disabled while `loading.isAddingTask`
  - Form cleared on successful task addition
- **ToDoList** (`molecules/toDo/toDoList.tsx`): Task list with drag-drop support via @dnd-kit
- **DatePicker** (`atoms/DatePicker.tsx`): Calendar popup wrapping react-day-picker + Popover
  - Quick-select buttons: Today, Tomorrow
  - Selecting already-selected date clears it
  - Locale-aware via date-fns locales
  - Disables past dates
- **useToDo hook**: Manages input state (title, priority, deadline), task filtering/grouping/sorting (via utilities), drag-drop reordering, error handling with toast notifications
  - Uses `taskSortUtils.sortTasks()` for filtering tasks by sort criteria
  - Uses `taskGroupUtils.groupTasks()` for grouping tasks by priority/date/due date
  - Uses `taskReorderUtils` for drag-drop order calculations and cross-group property updates
  - Reduced from ~500 to ~350 lines after utility extraction (Phase 1)
- **todoStore**: Zustand store with task lists, tasks, CRUD operations, loading states, error handling, persisted via Chrome storage

### Task Drag & Drop
Built with @dnd-kit for task reordering within and across groups:

**Components:**
- `SortableTaskItem`: Wraps TaskItem with `useSortable` hook
- `DroppableGroup`: Group container with `useDroppable` + separate `SortableContext`
- `TaskDragOverlay`: Visual feedback during drag (currently disabled)

**Sort Behavior:**
| SortBy | Drag Behavior |
|--------|---------------|
| MANUAL | Full control via `order` field |
| PRIORITY | Reorder within priority; cross-group changes `task.priority` |
| DUE_DATE | Reorder within deadline status; drop to "No Date" clears deadline |
| DATE/TITLE | Auto-switches to MANUAL on drag |

**Cross-Group Drag Effects:**
- Drag to priority group → updates `task.priority`
- Drag to "No Date" group → clears `task.deadline`
- Drag to/from Completed → toggles `task.isCompleted`

**Key Implementation Details:**
- Separate `SortableContext` per group to isolate reorder animations
- Custom collision detection using `pointerWithin` for accurate group targeting
- `disableSortAnimation` prop prevents swap visuals during cross-group drags
- Fractional ordering (insert between existing orders) minimizes order recalculation

### Component Styling
- Glass-morphism: `backdrop-blur-sm bg-gray-400/25 border-white/10`
- Z-index layers: 0 (blur bg), 1 (overlay), 2 (canvas), 10 (widgets), 50 (sidebar), 100 (toasts)
- Transitions: `transition-all duration-200`

## Environment Variables

```
VITE_WIKIART_ACCESS_CODE=<access_code>
VITE_WIKIART_SECRET_CODE=<secret_code>
```

Note: Firebase config is currently hardcoded in `firestoreService.ts`.

## Commands

```bash
npm run dev      # Vite dev server (CRX plugin disabled)
npm run build    # Build to dist/
npm run lint     # ESLint
npm run preview  # Preview production build
```

## Adding Features

**New Widget:**
1. Add to `WidgetId` enum in `types/settings.ts`
2. Create widget state interface extending `BaseWidgetState`
3. Add to `WidgetStates` interface and `DEFAULT_SETTINGS`
4. Register in `WIDGET_REGISTRY` in `constants/widgets.ts`
5. Create widget component (check `enabled && visible`)
6. Add settings fields to `settingConfig.ts`

**New Setting:**
1. Add to appropriate section in `UserSettings` interface
2. Add default in `DEFAULT_SETTINGS`
3. Add field config in `settingConfig.ts` (use dot notation: `widgets.clock.enabled`)
4. Add translation keys in `locales/*.json`

**New Museum API:**
1. Create service in `services/api/`
2. Add museum type to `Artwork` interface
3. Update fallback chain in `useArtwork.ts`
4. Add to `ArtworkSettings.museum` type

**New Language:**
1. Create `locales/<lang>.json`
2. Add to `i18n.ts` resources
3. Add to `Language` enum in `constants/common.ts`
4. Add selector option in `settingConfig.ts`

**Adding Error Handling to New Store Actions:**
1. Add loading state key to `TodoLoadingState` interface (if applicable)
2. Wrap action in try/catch/finally block
3. Snapshot current state before mutation (for rollback)
4. Set loading state to `true` before operation
5. Perform optimistic update + backend persistence
6. On error: rollback state, log error, rethrow to hook
7. In finally: set loading state to `false`
8. In hook handler: catch error, show toast with retry button
9. Add translation keys for success/error messages

## Architecture Notes

- No background/content scripts - pure new tab UI
- InteractiveArtwork uses HTML5 Canvas with grid-based dissolve effect
- Mouse tracking with physics (velocity, friction, ease)
- Animation stops when idle for performance
- 3-level image URL fallbacks for reliability
- Settings migrate automatically from legacy flat structure to new grouped structure
- **Error Handling**: All async operations have try/catch with rollback on failure
  - Optimistic updates with automatic rollback prevent UI inconsistency
  - User feedback via toast notifications for all actions
  - Loading states prevent race conditions and double-clicks
  - Real-time listener handles eventual consistency after rollback

## Refactoring Journey (3 Phases)

**Phase 1: Extract Complex Logic** ✅
- Extracted 280 lines of filtering, grouping, and reordering logic from `useToDo` hook
- Created 3 utility modules: `taskSortUtils`, `taskGroupUtils`, `taskReorderUtils`
- Hook reduced from 500 to 350 lines
- Pure functions enable unit testing in isolation
- Clear separation between UI orchestration and business logic

**Phase 2: Optimize Service Layer** ✅
- Added granular methods to `TodoService` interface (10 new methods)
- `LocalTodoService`: In-memory cache eliminates redundant reads
- `FirestoreTodoService`: Direct operations (`setDoc`, `updateDoc`, `deleteDoc`) instead of batch writes
- **99.5% reduction** in Firestore operations (201 ops → 1 op for adding a task with 100 existing)
- Batch methods preserved for legitimate bulk operations (migration, bulk delete, reordering)
- Updated 15 store actions to use granular methods

**Phase 3: Error Handling & User Feedback** ✅
- Created toast notification system with auto-dismiss and action buttons
- Added error handling to all 15 async store actions (try/catch/finally pattern)
- Implemented rollback on failure (snapshot state → optimistic update → rollback on error)
- Added loading states to prevent race conditions and double-clicks
- Updated hook handlers to show success/error toasts with retry capability
- Zero silent failures: all errors caught, logged, and displayed to user
- Toast UX: max 3 visible, debounced duplicates, glass-morphism styled

**Impact:**
- **Maintainability**: Clear separation of concerns, testable utilities, consistent error patterns
- **Performance**: 99.5% reduction in Firestore costs, no redundant storage reads
- **User Experience**: Immediate feedback, retry on errors, loading states, zero silent failures
- **Code Quality**: From 500-line monolithic hook to modular, well-tested architecture

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RyeNguyen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
