## elemental-website

> - DRY (Don't Repeat Yourself): Check for existing utilities before creating new ones

# Elemental Website - Code Standards

## General Principles
- DRY (Don't Repeat Yourself): Check for existing utilities before creating new ones
- Single Responsibility: Each component/function should do one thing well
- Type Safety: Always use TypeScript types, avoid `any` unless absolutely necessary

## Component Standards

### Size Limits
- **Maximum 200 lines** per component file
- If a component exceeds 200 lines, split it into sub-components
- Extract complex logic into separate utility functions or hooks

### File Organization
- Components go in `src/components/`
- Page-specific components go in `src/app/(frontend)/[page]/components/`
- Utilities go in `src/utilities/`
- No logic in page files - they should orchestrate components only

### Inline Styles
- **NEVER use inline styles** except for dynamic values from database (colors, positions)
- Use CSS classes from SCSS files
- For admin panel: Use classes from `src/app/(payload)/styles/`
- For frontend: Use Tailwind or `src/app/(frontend)/globals.css`

## Code Reuse Patterns

### Icons
- **Data-driven icons** (from database): Use utilities from `@/utilities/roleIcons`
  - `getGameRoleIcon()` for player roles
  - `getOrgRoleIcon()` for staff roles
  - `getProductionIcon()` for production roles
- **UI decoration icons**: Direct import from `lucide-react`

### API Routes
- Use `@/utilities/apiAuth` for authentication
  - `authenticateRequest()` for auth
  - `apiErrorResponse()` for errors
  - `apiSuccessResponse()` for success
- Always check authentication before processing

### Admin Components
- Use `@/utilities/adminAuth` for role checks
  - `useIsAdmin()`, `useIsTeamManager()`, etc.
- Use `@/utilities/adminHooks` for data fetching
  - `useAssignedTeams()`, `useDashboardStats()`, etc.
- Use `@/utilities/personHelpers` for safe person data extraction
  - `isPopulatedPerson()`, `getPersonNameFromRelationship()`, etc.

### Date/Time Formatting
- Use `@/utilities/formatDateTime` for all date/time operations
- Available functions: `formatDateOnly()`, `formatTimeOnly()`, `formatDateRange()`, `formatRelativeTime()`

## CSS Architecture (Admin Panel)

### File Structure
- Component styles go in `src/app/(payload)/styles/components/_component-name.scss`
- **Max 500 lines** per SCSS file
- Always import new files in `custom.scss`

### Variables & Mixins
- Use existing variables from `_variables.scss` (e.g., `$admin-accent-primary`)
- Use existing mixins from `_mixins.scss` (e.g., `@include gradient-border`)
- Check before creating new variables/mixins

### Specificity
- **Maximum 3 levels** of nesting in SCSS
- Avoid `!important` unless overriding third-party styles
- Use BEM-style class names for clarity

### Pattern Recognition
- If a pattern repeats **3+ times**, create a mixin
- If a value repeats **3+ times**, create a variable
- Document the purpose in comments

## TypeScript Best Practices

### Type Guards
- Use type guards from `@/utilities/personHelpers` for relationship handling
- Create type guards for complex data structures
- Never assume data shape without checking

### Async/Await
- Always use try/catch for async operations
- Provide meaningful error messages
- Log errors in development, hide details in production

### Null Safety
- Check for null/undefined before accessing properties
- Use optional chaining (`?.`) for nested properties
- Provide fallback values with nullish coalescing (`??`)

## Performance

### List Components
- Use `@/utilities/peopleListDataCache` for data shared across rows
- Don't fetch data per row - fetch once and share
- Use `memo()` for expensive list cell components

### Code Splitting
- Keep page files small (orchestration only)
- Split large components into smaller focused components
- Use dynamic imports for heavy components if needed

## Admin Panel Specifics

### Custom Pages with Sidebar
- Create as **Payload Globals** with UI field type
- Hide unnecessary controls with `@/components/EmptyComponent`
- Remove duplicate titles (Global provides the title)

### Custom Views in Collections
- Use custom cell components for list columns
- Use custom field components for edit views
- Reference components as string paths in config

## Documentation

### When to Document
- Complex algorithms or business logic
- Non-obvious workarounds
- Integration patterns
- Why something was done (not what - code shows what)

### Where to Document
- Inline comments for complex code
- README files for major features/patterns
- JSDoc for public utilities and hooks
- `docs/` folder for architectural decisions

## Pre-Commit Checklist
- [ ] No files exceed size limits (200 lines component, 500 lines SCSS)
- [ ] No new inline styles (except dynamic values)
- [ ] No duplicate code (checked for existing utilities)
- [ ] All TypeScript errors resolved (run `npm run type-check`)
- [ ] Used existing variables/mixins where applicable
- [ ] No `!important` added unnecessarily
- [ ] Component names are descriptive and follow patterns

**Note:** A pre-push git hook automatically runs `npm run type-check` before allowing pushes.
To set up hooks after cloning, run: `./scripts/setup-git-hooks.sh`

## Migration Notes
As of December 21, 2025:
- Phase 1-3 refactoring complete (frontend components)
- Phase 4A-D refactoring complete (admin panel)
- 1,000+ lines of duplicate code eliminated
- 390+ inline styles converted to CSS
- All audit findings addressed

Maintain these standards going forward!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Volence) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
