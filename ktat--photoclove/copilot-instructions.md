## photoclove

> 以下は過去のセッション分析から最も頻繁に指摘される問題です：

# CLAUDE.md - PhotoClove Development Guidelines

## ⚠️ 最頻出の問題 TOP5（必ず最初に確認）

以下は過去のセッション分析から最も頻繁に指摘される問題です：

1. **CSS変数を使う** - ハードコードNG
   - ❌ `color: '#666'` → ✅ `var(--color-text-muted)`
   - ❌ `fontSize: '14px'` → ✅ `var(--font-size-base)`
   - ❌ `padding: '8px'` → ✅ `var(--space-2)`
   - **ESLint設定済み**: `pnpm lint` で自動検出
   - **pre-file-writeフック**: ファイル保存時に自動チェック

2. **600行制限を守る** - ファイルが大きくなったら分割
   - 機能ごとにモジュール分割を検討
   - 大きなコンポーネントはサブコンポーネントに分解

3. **ログは構造化** - console.log / println! は使わない
   - ❌ `console.log('error', data)` → ✅ `logger.error('Component', 'event', 'msg', data)`
   - ❌ `println!("error: {}", msg)` → ✅ `log::error!(target: "module", "event; msg={}", msg)`

4. **既存utilsを使う** - 実装前に必ず確認
   - FileUtils, DateUtils, StringUtils, PathUtils を確認
   - Unified Search API があるなら個別検索実装しない
   - 共通コンポーネント（Modal, Loading等）を再作成しない

5. **状態は不変に** - Reactの基本原則
   - ❌ `config.items.push(item)` → ✅ `setConfig(prev => ({...prev, items: [...prev.items, item]}))`
   - ❌ 関数内で `let lock = false` → ✅ `const lockRef = useRef(false)`

詳細な問題リスト（TOP20）は `docs/common-mistakes.md` を参照してください。

## 📋 General Development Rules (Apply to ALL Tasks)

### 1. Logging Standards

When implementing logging in the codebase:

#### Frontend Logging

- Never use console.log
- Use the structured `logger` service from `src/services/LoggerService.js`
- Import: `import { logger } from '../services/LoggerService.js';`
- Pattern: `logger.level('ComponentName', 'event_name', 'Description', { data })`
- Example: `logger.info('PhotosList', 'search_triggered', 'User initiated search', { query, filters })`
- Avoid direct `console.log/warn/error` calls - use the structured logger instead

#### Backend Logging
- Nerver use `print!`, `println!` etc.
- Use Rust's `log` macros with structured format in semicolon-separated key=value pairs
- Pattern: `log::level!(target: "component", "event; key1={}; key2={}", value1, value2)`
- Always include `correlation_id` when available for request tracing
- Example: `log::info!(target: "search", "search_request; correlation_id={}; query={}", correlation_id, query)`

#### LogViewer Integration
- All logs are automatically collected and viewable in LogViewer.jsx
- Frontend logs: stored in memory via LoggerService
- Backend logs: written to daily files and retrieved via `get_logs` command
- Use structured logging to enable proper filtering and search in LogViewer

### 2. Code Style & Patterns

#### Terms & Source Code Reference
For effective communication about PhotoClove features and implementation:

- **Use terms document**: Refer to `docs/terms.md` for standard terminology and source code mappings
- **Find implementation quickly**: Use the Term → File mapping to locate where features are implemented
- **Consistent naming**: Use the documented terms when discussing features, components, and concepts
- **Example**: Instead of saying "photo grid component", use "PhotosList" and reference `src/App/PhotosList.jsx`

When working on features:
1. Check `docs/terms.md` for the correct term and file location
2. Use the documented patterns (e.g., `toggle*()` for UI state, `use*()` for hooks)
3. Follow the source code structure shown in the mappings

#### UI Theme & Color Guidelines
**PhotoClove is a dark theme application**. Follow these strict color rules:

- **Light colors prohibition**: NEVER use light colors (white, light gray) for large areas or backgrounds
- **Light color usage**: Light colors can ONLY be used for small accents, active states, focus indicators
- **Hardcoded colors to avoid**: Never use `white`, `#fff`, `#ffffff`, `#f5f5f5`, `#fafafa` for backgrounds
- **Always use CSS variables**: Never hardcode colors or sizes - use the design system tokens

#### CSS Design System (src/styles/base.css)
All styling must use CSS variables defined in `base.css`. **Never hardcode colors, font sizes, or spacing values.**

**Background Colors:**
| Variable | Usage |
|----------|-------|
| `--color-bg-base` | Main page background |
| `--color-bg-elevated` | Cards, modals, elevated surfaces |
| `--color-bg-surface` | Input fields, interactive surfaces |
| `--color-bg-muted` | Disabled states, subtle backgrounds |

**Text Colors:**
| Variable | Usage |
|----------|-------|
| `--color-text-primary` | Main text, headings |
| `--color-text-secondary` | Secondary text, descriptions |
| `--color-text-muted` | Placeholder, disabled text |

**Border Colors:**
| Variable | Usage |
|----------|-------|
| `--color-border-default` | Standard borders |
| `--color-border-subtle` | Subtle dividers |
| `--color-border-strong` | Emphasized borders |

**State Colors:**
| Variable | Usage |
|----------|-------|
| `--color-primary` | Primary actions, links |
| `--color-primary-hover` | Primary hover state |
| `--color-primary-selected` | Selected item background |
| `--color-success` | Success states |
| `--color-warning` | Warning states, stars |
| `--color-danger` | Error states, delete actions |

**Font Sizes:**
| Variable | Size | Usage |
|----------|------|-------|
| `--font-size-xs` | 9px | Very small indicators |
| `--font-size-xs` | 11px | Small labels, metadata |
| `--font-size-sm` | 13px | Secondary text |
| `--font-size-base` | 14px | Body text (default) |
| `--font-size-lg` | 16px | Subheadings, emphasis |
| `--font-size-xl` | 18px | Section titles |
| `--font-size-2xl` | 20px | Page titles |

**Spacing (4px base):**
- `--space-1` (4px), `--space-2` (8px), `--space-3` (12px), `--space-4` (16px), `--space-5` (20px), `--space-6` (24px)

**Border Radius:**
- `--radius-sm` (4px), `--radius-md` (6px), `--radius-lg` (8px), `--radius-xl` (12px)

**Example - Before/After:**
```javascript
// ❌ BAD: Hardcoded values
style={{ fontSize: '14px', color: '#666', backgroundColor: '#374151' }}

// ✅ GOOD: CSS variables
style={{ fontSize: 'var(--font-size-base)', color: 'var(--color-text-muted)', backgroundColor: 'var(--color-bg-muted)' }}
```

**Enforcement:**
- **ESLint**: Run `pnpm lint` to detect all hardcoded pixel values
- **Pre-write Hook**: Automatic detection when files are saved via Claude
- **Documentation**: See `docs/eslint-hardcoded-pixels.md` for full mapping guide

#### CSS Modules Guidelines
PhotoClove uses CSS Modules for component-specific styling. Follow these guidelines:

- **New components**: Always use CSS Modules for new component styles
- **Import pattern**: `import styles from './Component.module.css';`
- **Class names**: Use `styles.className` or `styles['class-name']` for kebab-case
- **Conditional classes**: Use `classnames` library for conditional class composition
  ```javascript
  import classNames from 'classnames';
  className={classNames(styles.base, { [styles.active]: isActive })}
  ```
- **File naming**: Use `ComponentName.module.css` naming convention
- **Global selectors in CSS Modules**: Use `:global()` when referencing external global classes
  ```css
  /* Reference global classes from child components */
  .preferences-section :global(.row2) { ... }
  /* Reference parent container classes */
  :global(.inner-container.left-menu-collapsed) .preferences { ... }
  ```
- **Global CSS exceptions**: Keep complex layout CSS (e.g., `PhotosList.css`) as global when:
  - CSS contains complex responsive rules with many media queries
  - Styles are shared across multiple components
  - Migration cost outweighs benefits
- **Migration priority**: Existing unmigrated files (search/, modals, utilities) are low priority - migrate only when actively modifying those components

#### Opening External URLs (Tauri)
In Tauri apps, standard `<a href="..." target="_blank">` links do NOT open in the browser. Use `openUrl` from the opener plugin:

```javascript
import { openUrl } from '@tauri-apps/plugin-opener';

// Open external URL in system default browser
openUrl('https://example.com');

// In onClick handler
<span onClick={() => openUrl('https://example.com')}>Link</span>
```

**Important**: Never use `target="_blank"` for external links - it won't work in Tauri.

### 3. Testing & Validation
- Run `cargo check` for Rust changes in `src-tauri/src/`
- Verify with appropriate test commands
- Check neighboring files for conventions and patterns

### 4. Code Quality Standards
- **DDD Architecture**: Prioritize Domain-Driven Design - separate business logic into domain layers, use repositories for data access, and maintain clear boundaries between layers
- **DRY Principle**: Don't Repeat Yourself - extract common logic into reusable functions, components, or modules
- **File Length Limit**: Keep each file under 600 lines - split large files into smaller, focused modules
- **Task Verification**: After implementing code changes, compare the implementation with the original task requirements. If any features are missing or incomplete, create subtasks to address them

### 5. Important Reminders
- Do what has been asked; nothing more, nothing less.
- NEVER create files unless they're absolutely necessary for achieving your goal.
- ALWAYS prefer editing an existing file to creating a new one.
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

## 🔧 Context-Specific Guidelines

### Database Migration Implementation

#### Database Migration System
**Directory**: `src-tauri/src/repository/meta_db/migrations/`

##### Migration Pattern
PhotoClove uses a versioned migration system with SQL files. All new tables should be added as new migration files.

**Migration Files**:
- `001_initial_schema.sql` - Core photo_metadata table
- `002_create_date_summary.sql` - Date summary for performance
- `003_create_collections.sql` - Albums and tags (unified collections)
- `004_create_job_queue.sql` - Background job queue

**Adding New Migrations**:
1. Create a new SQL file: `NNN_description.sql`
2. Register in `migrations/mod.rs`:
```rust
Migration {
    version: N,
    name: "description",
    sql: include_str!("NNN_description.sql"),
},
```

##### Collections Tables (Albums & Tags)
**File**: `003_create_collections.sql`

**Tables**:
1. **`photo_collections`**: Unified storage for albums and tags
   - `type`: 'album' or 'tag'
   - `name`, `color`, `description`, `cover_photo_path`
   - `settings` (JSON), timestamps

2. **`photo_collection_items`**: Junction table for collection-photo relationships
   - `collection_id`, `photo_path`, `order_index`, `added_at`
   - `metadata` (JSON for additional data)

**Key Features**:
- **Versioned migrations**: Tracked in `_migrations` table
- **Automatic execution**: `run_migrations()` applies pending migrations on startup
- **Legacy support**: Handles old schema migration automatically
- **Unified design**: Albums and tags share the same table structure

##### Best Practices
When adding new tables:

1. **Create migration file**: Add numbered SQL file in migrations directory
2. **Register migration**: Add to MIGRATIONS array in `mod.rs`
3. **Use proper constraints**: Foreign keys, indexes, CHECK constraints
4. **Handle errors**: Migrations use Result types
5. **Test thoroughly**: Verify migration works on fresh and existing databases

### Bug Investigation

#### Investigation Process
1. **Check logs first**: `~/.local/share/photoclove/logs/`
2. **Trace data flow**: User Action → State → Backend → Response → UI
3. **Use optional chaining** for null-safe property access
4. **Investigate systematically**, avoid repeated fixes

### Debugging Full-Stack Issues

#### Key Principles (Learned from Date Count Bug - 2025-12-22)

When debugging issues that span frontend and backend:

##### 1. **Don't Assume One Side is Correct**
- ❌ BAD: "Backend works, fix the frontend"
- ✅ GOOD: Investigate both frontend AND backend
- **Example**: Date count update issue was in backend date calculation, not frontend state management

##### 2. **Compare Working vs Non-Working Code**
- If user says "X works but Y doesn't", **compare their implementations carefully**
- **Example**: `move_to_trash` (single, working) vs `move_to_trash_batch` (batch, broken)
- Look for differences in:
  - How data is retrieved
  - How dates/values are calculated
  - What parameters are passed

##### 3. **Pay Attention to Log Inconsistencies**
- Date/time format mismatches (e.g., `2025-12-22` vs `2023/05/04`)
- Value type differences (string vs number, different delimiters)
- **These are red flags pointing to the real problem**

##### 4. **Prefer Simplification Over Addition**
- ❌ BAD: Add more handlers, props, state management
- ✅ GOOD: Find and fix the root cause
- **Before adding code, ask**: "Can I fix this by simplifying existing code?"

##### 5. **Check Entity/Domain Logic First**
- Issues with data often originate in domain entities (Photo, Date, etc.)
- Check how entities calculate/derive their properties
- **Example**: Photo's date calculation method had inconsistencies

#### Debugging Checklist for "Feature X Not Working"

1. **Identify Working vs Non-Working Scenarios**
   - What works? What doesn't?
   - Compare their code paths

2. **Check Both Ends**
   ```
   Frontend: State → Handler → API Call
   Backend: Command → Entity → Repository → Database
   ```

3. **Look for Format/Type Mismatches**
   - Check logs for inconsistent data formats
   - Verify data types match expectations

4. **Find the Simplest Fix**
   - Can you fix it in the entity/domain layer?
   - Can you unify duplicated logic?
   - Can you remove code instead of adding?

5. **Test the Theory**
   - Add targeted console.log/log::debug to verify hypothesis
   - Compare actual vs expected values at each step

#### When to Check Backend vs Frontend

**Check Backend When:**
- Data format inconsistencies in logs
- Single operation works but batch doesn't (or vice versa)
- Issue involves dates, calculations, or data transformations
- Same frontend code worked before (backend might have changed)

**Check Frontend When:**
- UI doesn't update despite correct API responses
- State management issues (useEffect, deps arrays)
- Event handlers not firing
- Props not passed correctly

**Check Both When:**
- Complete feature not working
- Unclear where the problem originates
- Logs show data flowing but results are wrong

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ktat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ktat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
