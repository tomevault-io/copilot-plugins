## nv-internal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ CRITICAL SECURITY WARNING

**THIS IS A PUBLIC REPOSITORY - NEVER COMMIT SECRETS OR API KEYS**

- ❌ **NEVER** push API keys, credentials, or secrets to this repository
- ❌ **NEVER** include real environment variable values in code or documentation
- ❌ **NEVER** commit `.env` files or files containing sensitive data
- ✅ **ALWAYS** use placeholder values in examples (e.g., `"your-api-key-here"`, `"pk_live_xxxxx"`)
- ✅ **ALWAYS** remind users to add secrets via GitHub Secrets UI or environment variables
- ✅ **ALWAYS** use `.env.example` files with placeholder values only
- ✅ **ALWAYS** verify that committed files don't contain real credentials before pushing

**If you discover committed secrets**:
1. Immediately alert the user
2. Remove the secrets from the code
3. Recommend rotating the compromised credentials
4. Use `git filter-branch` or BFG Repo-Cleaner to remove from history if needed

## Project Overview

NV Internal is a task management application for an air conditioning service company in Vietnam. This is a monorepo with pnpm workspaces containing a REST API and React Native mobile app.

### Core Architecture

- **API**: Hono-based REST API with PostgreSQL database, deployed on Vercel
- **Mobile**: Expo React Native app with Clerk authentication
- **Database**: PostgreSQL via Neon (serverless) with local Docker option
- **Authentication**: Clerk for user management
- **Packages**: Shared Prisma client and validation schemas

## 📚 Documentation Structure

Detailed documentation is organized in the `docs/` directory:

- **[Architecture Patterns](./docs/architecture/patterns/)** - Implementation patterns and best practices
- **[Development Guide](./docs/development/)** - Setup, commands, and workflows
- **[Testing Guide](./docs/testing/)** - Testing strategies and patterns

## Specialized Agent Usage

**IMPORTANT**: Claude Code should **ALWAYS prefer using specialized agents** for development tasks. Agents are purpose-built experts with deep domain knowledge and should be the default choice for implementation work.

### General Agent Usage Philosophy

**Default Behavior**:
- ✅ **USE AGENTS**: For all backend, frontend, and quality assurance tasks
- ✅ **USE AGENTS**: When implementing features, fixing bugs, or making architectural changes
- ❌ **Don't handle directly**: Complex implementation tasks without agent expertise
- ❌ **Don't handle directly**: Multi-step development work that requires specialized knowledge

**When to use agents vs. handling directly**:
- **Use agents**: Implementation, debugging, architecture, testing, documentation
- **Handle directly**: Simple file reads, answering questions, explaining code, quick fixes

### Backend Development Agent

**Agent**: `backend-engineer`

**USE THIS AGENT FOR ALL BACKEND WORK**

**Capabilities**:
- API endpoint implementation and architecture
- Database schema design and Prisma operations
- Service layer business logic
- Hono route configuration and middleware
- Clerk authentication integration
- Backend testing with Jest
- System architecture decisions
- Performance optimization
- Security reviews

**When to invoke (ALWAYS for backend work)**:
- ✅ Implementing new API features or endpoints
- ✅ Designing database migrations or schema changes
- ✅ Debugging backend issues or authentication problems
- ✅ Reviewing backend code for best practices
- ✅ Writing or updating service layer functions
- ✅ Creating or modifying Hono routes
- ✅ Working with Prisma models or queries
- ✅ Adding or updating validation schemas

**Project-Specific Context for backend-engineer**:

The backend-engineer agent is configured with generic backend development expertise. When working on this project, it needs to be aware of these NV Internal-specific details:

**Technology Stack**:
- **Framework**: Hono-based REST API
- **Database**: PostgreSQL via Neon (serverless)
- **ORM**: Prisma
- **Authentication**: Clerk with @hono/clerk-auth middleware
- **Deployment**: Vercel serverless functions
- **Validation**: Zod schemas in `@nv-internal/validation` package
- **Testing**: Jest with ts-jest preset, mock-based (no real database)

**Project Architecture**:
- **Monorepo Structure**: pnpm workspaces with apps/api, apps/mobile, and shared packages
- **API Routes**: Located in `apps/api/src/v1/` with service layer pattern
- **Shared Packages**:
  - `@nv-internal/validation` - Zod validation schemas
  - `@nv-internal/prisma-client` - Shared Prisma client
- **Database Schema**: Task management system with prefixed IDs (cust*, geo*, act_*, pay_*)
- **Authentication**: All routes require Clerk middleware; no public endpoints
- **Testing**: Tests in `__tests__/` directories alongside source files

**Key Implementation Patterns**:
- Use prefixed IDs for all models (follow patterns: cust*, geo*, act_*, pay_*)
- Log all state changes to Activity table for audit trails
- Use Prisma transactions for multi-model operations
- Place business logic in `*.service.ts` files, not route handlers
- Define Zod schemas in `@nv-internal/validation` for type safety
- Follow cursor-based pagination for lists
- Use Clerk JWT claims for optimized authentication (see Auth Optimization pattern)

**Quality Standards**:
- Run `pnpm biome:check --write .` before committing
- Ensure all tests pass: `pnpm --filter @nv-internal/api test`
- Use conventional commit format: `type(scope): description`
- Rebuild shared packages if modified:
  - `pnpm --filter @nv-internal/prisma-client build`
  - `pnpm --filter @nv-internal/validation build`

**Reference Documentation**:
- Architecture patterns: `docs/architecture/patterns/`
- Testing patterns: `docs/testing/` and `apps/api/README.md#testing`
- V1 feature plans: `.claude/plans/v1/`

### Frontend Development Agent

**Agent**: `frontend-engineer`

**USE THIS AGENT FOR ALL MOBILE/UI WORK**

**Capabilities**:
- React Native component implementation
- Expo features and mobile-specific functionality
- Mobile UI/UX design and implementation
- Navigation and routing with Expo Router
- TanStack Query state management
- NativeWind styling and responsive design
- Mobile app optimization
- API integration from mobile app
- Form handling and validation

**When to invoke (ALWAYS for frontend work)**:
- ✅ Building new mobile screens or components
- ✅ Implementing mobile-specific features
- ✅ Debugging mobile UI or navigation issues
- ✅ Optimizing mobile app performance
- ✅ Integrating with backend APIs
- ✅ Styling with NativeWind/Tailwind
- ✅ Managing client-side state with TanStack Query
- ✅ Working with Expo Router navigation

### Code Quality Agent

**Agent**: `code-quality-enforcer`

**USE THIS AGENT AFTER IMPLEMENTATION OR BEFORE COMMITS**

**Capabilities**:
- Validating code changes before commits
- Running pre-commit checks (format, lint, tests)
- TypeScript compilation checking and fixing
- Ensuring adherence to project standards
- Running targeted tests on changed files
- Building shared packages if modified

**When to invoke (ALWAYS after changes)**:
- ✅ After completing implementation work
- ✅ Before creating commits or pull requests
- ✅ When preparing code for review
- ✅ After fixing bugs or making changes
- ✅ When TypeScript errors need resolution

**Quality checks performed**:
1. TypeScript compilation: `npx tsc --noEmit` (fix all TS errors)
2. Format and lint: `pnpm exec biome check --write .`
3. Tests on changed code:
   - If API changes: `pnpm --filter @nv-internal/api test` (specific tests if possible)
   - If mobile changes: Test relevant screens/components only
4. Build shared packages if modified:
   - `pnpm --filter @nv-internal/prisma-client build`
   - `pnpm --filter @nv-internal/validation build`

**Important**: Only test changed files/modules unless full codebase testing is explicitly requested

### Documentation Tracking Agent

**Agent**: `task-doc-tracker`

**USE THIS AGENT FOR ALL DOCUMENTATION WORK**

**Capabilities**:
- Creating and maintaining task documentation in `.claude/tasks/`
- Tracking implementation progress and requirements
- Updating v1 feature plans in `.claude/plans/v1/`
- Extracting learnings and updating documentation
- Ensuring project knowledge is preserved
- Linking tasks to feature plans
- Documenting architectural decisions

**When to invoke (PROACTIVELY)**:
- ✅ Starting new feature development
- ✅ Completing implementations
- ✅ After significant code changes
- ✅ Before and after implementing v1 planned features
- ✅ When architectural decisions are made
- ✅ When patterns should be documented
- ✅ When updating enhancement ideas

**Documentation flow**:
1. Plan feature → Create task file
2. Implement → Update progress in task file
3. Complete → Mark as ✅, update v1 plan status
4. Extract learnings → Update documentation with patterns

### Mobile QA Testing Agent

**Agent**: `qa-ui`

**USE THIS AGENT FOR MOBILE UI TESTING**

**Capabilities**:
- Creating comprehensive test scenarios for mobile features
- Executing tests using Mobile-MCP tools
- Documenting test results and bugs found
- Identifying UI/UX issues and edge cases
- Verifying accessibility compliance
- Performance testing for mobile interactions
- Cross-device compatibility testing

**When to invoke (ALWAYS for mobile testing)**:
- ✅ Testing new mobile features before release
- ✅ Verifying bug fixes in the mobile app
- ✅ Running regression tests after changes
- ✅ Checking accessibility of UI components
- ✅ Validating user workflows end-to-end
- ✅ Testing edge cases and error scenarios
- ✅ Performing performance testing on lists/animations

**QA testing workflow**:
1. Review test plan in `.claude/qa/test-plans/`
2. Create detailed scenarios in `.claude/qa/test-scenarios/`
3. Execute tests using Mobile-MCP tools
4. Document results in `.claude/qa/test-results/`
5. Report bugs with reproduction steps
6. Verify fixes and update test status

**Test documentation structure**:
- **Test Plans**: `.claude/qa/test-plans/` - Feature-level test specifications
- **Test Scenarios**: `.claude/qa/test-scenarios/` - Step-by-step test cases
- **Test Results**: `.claude/qa/test-results/` - Execution results and bug reports
- **Mobile Testing Guide**: `.claude/qa/mobile-testing-guide.md` - Mobile-MCP usage

### Agent Workflow Example

**Typical feature implementation flow**:

1. **Documentation Setup**: Launch `task-doc-tracker` to create task file
2. **Implementation**: Launch `backend-engineer` or `frontend-engineer` for implementation
3. **Code Quality**: Launch `code-quality-enforcer` to verify changes
4. **Mobile Testing**: Launch `qa-ui` to test mobile features (if applicable)
5. **Documentation Update**: Launch `task-doc-tracker` to mark complete and extract learnings

**Testing workflow for existing features**:

1. **Test Planning**: Review or create test plan in `.claude/qa/test-plans/`
2. **Test Execution**: Launch `qa-ui` to execute test scenarios
3. **Bug Reporting**: Document bugs in test results with reproduction steps
4. **Fix Implementation**: Launch appropriate expert agent to fix issues
5. **Verification**: Launch `qa-ui` to verify fixes
6. **Documentation**: Update test results and close bugs

## Key Architecture Patterns

For detailed patterns, see [Architecture Patterns](./docs/architecture/patterns/). Key patterns include:

- **[Route Organization](./docs/architecture/patterns/route-organization.md)** - RESTful route structure and mounting
- **[GPS Verification](./docs/architecture/patterns/gps-verification.md)** - Accurate distance calculations
- **[Activity-Based Events](./docs/architecture/patterns/activity-event.md)** - Unified event logging
- **[Error Handling](./docs/architecture/patterns/error-handling.md)** - HTTPException usage
- **[Authentication](./docs/architecture/patterns/auth-middleware.md)** - Security best practices
- **[Payment Transactions](./docs/architecture/patterns/payment-transactions.md)** - Serverless-safe transactions
- **[File Uploads](./docs/architecture/patterns/file-upload.md)** - Hono RPC limitations
- **[Cache Invalidation](./docs/architecture/patterns/cache-invalidation.md)** - TanStack Query patterns
- **[Timezone Handling](./docs/architecture/patterns/timezone-handling.md)** - Modern TZDate for accurate date boundaries
- **[Tabs Navigation](./docs/architecture/patterns/tabs-navigation.md)** - **CRITICAL**: Stable tabs implementation with haptic feedback
- **[Vietnamese Search](./docs/architecture/patterns/vietnamese-search.md)** - Accent-insensitive search for Vietnamese text
- **[SearchableText](./docs/architecture/patterns/searchable-text.md)** - Pre-computed search fields for performance
- **[OTA Updates](./docs/architecture/patterns/ota-updates.md)** - Hook-Only pattern for Expo Updates with graceful degradation
- **[Mobile Accessibility](./docs/architecture/patterns/mobile-accessibility.md)** - Required accessibility properties for all interactive elements (MobileMCP compatibility)
- **[Feature Flags](./docs/feature-flags-guide.md)** - PostHog feature flags with graceful degradation pattern for controlled rollouts and instant kill switches

### Recently Established Patterns

#### Feature Flags Pattern (2025-11-07)

**PostHog Feature Flags**: Implemented controlled feature rollout system with 4 production flags demonstrating independent feature control.
- **Problem**: No mechanism for gradual feature rollouts, A/B testing, or instant kill switches
- **Solution**: PostHog feature flags with custom hooks, type-safe constants, and graceful degradation
- **Naming Convention**: `task-list-[feature]-enabled-[role]` format for consistency
- **Production Flags Implemented**:
  - `task-list-filter-enabled-admin` - Admin task list filtering UI
  - `task-list-search-enabled-admin` - Admin task list search functionality
  - `task-list-filter-enabled-worker` - Worker task list filtering UI
  - `task-list-search-enabled-worker` - Worker task list search functionality
- **Key Pattern**: No loading states for instant graceful degradation (features absent if disabled)
- **Flag Independence**: All 16 combinations tested - filter and search work independently
- **Impact**: Instant feature control without app deployments, role-based rollouts, A/B testing ready
- **Implementation Tasks**:
  - Initial: `.claude/tasks/20251107-043354-first-feature-flag-task-list-filter-admin.md`
  - Additional: `.claude/tasks/20251107-050000-implement-additional-task-list-feature-flags.md`
- **Documentation**: `.claude/docs/feature-flags-guide.md` (v1.1 with production examples)
- **Commits**: `b0d2576` (first flag), `9323b53` (three additional flags)

#### PostHog Provider Initialization Pattern (2025-11-07)

**Critical Production Fix**: Established correct PostHog Provider initialization pattern after production crash.
- **Problem**: App crashed immediately after splash screen in TestFlight builds
- **Root Cause**: Incorrect PostHogProvider prop pattern incompatible with posthog-react-native v4.x
- **Wrong Pattern**: Passing `apiKey` AND `options` as separate props to PostHogProvider
- **Correct Pattern**: Use `client` prop with PostHog instance created via `new PostHog(apiKey, options)`
- **Key Learning**: Always use Pattern 3 (client prop with useMemo) for complex configurations
- **Implementation**:
  ```typescript
  // Create client with useMemo to prevent recreation
  const client = React.useMemo(() => new PostHog(apiKey, options), [])

  // Pass client prop (NOT apiKey/options separately)
  <PostHogProvider client={client} autocapture={{ ... }}>
  ```
- **Impact**: Fixed CRITICAL production crash affecting all TestFlight users
- **Verification**: TypeScript passes, Biome clean, requires manual testing
- **Implementation Task**: `.claude/tasks/20251107-095033-fix-posthog-provider-crash.md`
- **Reference**: Official PostHog React Native Expo example

#### Mobile Accessibility Pattern (2025-11-06)

**MobileMCP Compatibility**: Established comprehensive accessibility standards for all interactive elements.
- **Problem**: 50% MobileMCP test failure rate, no screen reader support
- **Solution**: Required 4 properties for all interactive elements (accessibilityLabel, accessibilityHint, accessibilityRole, testID)
- **testID Convention**: `{screen}-{action}-{type}` format for reliable testing
- **Vietnamese Labels**: All accessibility labels in Vietnamese with proper grammar
- **Dynamic Labels**: Update based on state (loading, count, context)
- **Impact**: 95%+ click success rate, 100% screen reader compliance
- **Files Modified**: 10 files, 97+ properties added, 32+ elements improved
- **Implementation Task**: `.claude/tasks/20251106-190000-app-store-screenshot-capture-mobilemcp.md`
- **Pattern Documentation**: [Mobile Accessibility](./docs/architecture/patterns/mobile-accessibility.md)
- **Screenshot Playbook**: `.claude/qa/screenshot-capture-playbook.md`

#### OTA Updates Pattern (2025-11-06)

**Hook-Only Pattern**: Implemented Over-The-Air updates with version tracking using a standalone hook pattern.
- **Problem**: No version visibility or update mechanism for users
- **Solution**: Hook-Only pattern without provider for single-component state
- **Key Features**: Expo Go detection, automatic background checks, user-controlled reload
- **Vietnamese UI**: All update strings in Vietnamese for consistency
- **Persistence**: AsyncStorage for update state following existing patterns
- **Implementation Task**: `.claude/tasks/20251106-085928-app-version-tracking-ota-updates.md`
- **Pattern Documentation**: [OTA Updates](./docs/architecture/patterns/ota-updates.md)
- **Fixed Deprecation**: Changed `appOwnership` to `executionEnvironment` for Expo Go detection

#### Stable Tabs Migration (2025-10-30)

**Critical Migration**: Replaced unstable NativeTabs with stable Tabs component due to persistent UI responsiveness issues.
- **Problem**: NativeTabs (unstable-native-tabs) caused unresponsive UI on initial module load
- **Root Cause**: Component marked as unstable with race conditions in navigation state
- **Solution**: Migrated to stable Tabs from expo-router with haptic feedback
- **Additional Fix**: Continue avoiding `screenOptions` at Stack level (causes invisible overlays)
- **Migration Task**: `.claude/tasks/20251030-041902-migrate-nativetabs-to-stable-tabs.md`
- **Original Issue**: `.claude/tasks/20251030-025700-fix-worker-unresponsive-ui.md`
- **Pattern**: [Tabs Navigation](./docs/architecture/patterns/tabs-navigation.md)

#### Employee Summary Implementation (2025-10-30)

These patterns were established during the Employee Summary feature implementation:

- **Batch Query Pattern**: Replace N+1 queries with batch queries for aggregate reports
  - Query all data once, process in-memory for multiple users
  - Use PostgreSQL array operators (`hasSome`) for efficient filtering
  - Example: Employee summary reduced 100+ queries to 2-3 (see `.claude/tasks/20251029-145000-employee-report-monthly-summary-enhancement.md`)

- **FlatList Optimization**: High-performance mobile lists
  - Always use `FlatList` over `ScrollView` for large datasets
  - Implement `getItemLayout` for known item heights
  - Use virtualization props: `removeClippedSubviews`, `windowSize`, etc.
  - Target 60fps scrolling performance

- **Defensive API Responses**: Always provide fallbacks
  - Never assume data exists - use optional chaining and nullish coalescing
  - Provide default values for missing user data (names, emails, etc.)
  - Example: `user.firstName || ""` instead of assuming firstName exists

- **Tied Ranking Algorithm**: Equal values get equal ranks
  - When ranking by metrics (revenue, tasks), equal values should have the same rank
  - Sequence: 1, 1, 3 (not 1, 2, 3) for tied first place
  - Important for fair employee performance comparisons

- **Client-Side Search**: Instant filtering without API calls
  - Filter results in-memory for immediate response
  - Use `useMemo` to optimize filtering performance
  - Provide search highlighting for better UX

#### User Search & Bottom Sheet Patterns (2025-10-30)

Patterns established while fixing Vietnamese search and bottom sheet usability:

- **Vietnamese Accent-Insensitive Search**: Support typing without diacritics
  - Use `removeVietnameseAccents()` utility for search normalization
  - Apply to both search query and target text for matching
  - Critical for Vietnamese users who omit accents for typing speed
  - Implemented with Fuse.js for fuzzy matching and typo tolerance
  - Task: `.claude/tasks/20251030-045028-fix-user-search-and-bottom-sheet.md`

- **Bottom Sheet List Integration**: Use specialized components for proper gesture handling
  - Always use `BottomSheetFlatList` from `@gorhom/bottom-sheet` (never standard `FlatList`)
  - Set appropriate `index` prop for initial height (e.g., `index={1}` for 90% screen)
  - Place action buttons in `ListFooterComponent` to scroll with content
  - Prevents gesture conflicts between sheet drag and list scroll

- **Null-Safe String Operations**: Defensive programming for optional fields
  - Always use `(value || '').toLowerCase()` for potentially null strings
  - Prevents runtime crashes with missing data (e.g., phone numbers)
  - Apply consistently across all string transformations

- **Email Display Pattern**: Graceful truncation for long text
  - Use `numberOfLines={1}` with `ellipsizeMode="middle"` for emails
  - Shows beginning and end of address for better recognition
  - Prevents UI overflow in constrained spaces

#### Admin/Worker Search & Filter Implementation (2025-10-30)

Comprehensive search and filter functionality added to admin and worker task lists:

- **SearchableText Pattern**: Pre-computed search field optimization
  - 64% code reduction (140 lines → 50 lines)
  - 2-3x performance improvement with single indexed field
  - Perfect pagination accuracy (no post-processing)
  - Vietnamese accent-insensitive search built-in
  - Task: `.claude/tasks/20251030-110000-complete-phase3-phase4-search-filter-ui.md`

- **Native Header Search**: iOS/Android native search bars
  - Uses `headerSearchBarOptions` for platform-specific UI
  - Real-time search with debouncing
  - Seamless integration with infinite scroll

- **Comprehensive Filtering**: Status, date, assignee, and sort
  - Color-coded status chips matching existing badges
  - Quick date presets with custom calendar picker
  - Vietnamese search for assignee selection
  - Active filter display with dismissible chips

- **Bottom Sheet Patterns**: Fixed layout with sticky footer
  - Separate modals to avoid VirtualizedList nesting
  - Safe area padding with pb-safe utility
  - Proper gesture handling with specialized components

#### Task Comments Implementation Patterns (2025-10-31)

Patterns established during Task Comments feature implementation:

- **Activity-Based Features**: Use Activity model for any feature that logs events
  - Zero new tables needed - reuse existing Activity model
  - Automatic integration with activity feed
  - Built-in pagination and filtering support
  - Example: Task comments implemented with TASK_COMMENTED action
  - Task: `.claude/tasks/20251030-130330-implement-task-comments.md`

- **FormData Handling in Hono**: Proper multipart/form-data parsing
  - Always use `formData()` method, not `parseBody()` for multipart endpoints
  - Example: `const { comment, files } = c.req.valid('form')`
  - Support both JSON and FormData in same endpoint by checking content type
  - Pattern documented in task comments implementation

- **Dual-Mode API Calls**: Optimize for different data types
  - Use JSON for simple data (text-only comments)
  - Use FormData with native fetch for file uploads
  - Pattern: Check for files and switch modes dynamically
  ```typescript
  if (!photos || photos.length === 0) {
    return callHonoApi(...) // JSON mode
  } else {
    return fetch(...) // FormData mode
  }
  ```

- **Photo Picker UI Pattern**: Consistent mobile photo selection
  - Permission handling with settings redirect on denial
  - Camera single selection + Gallery multi-selection
  - Preview thumbnails with individual remove buttons
  - Horizontal ScrollView for photo list
  - Badge component for count indicator (e.g., "3/5")
  - Max limit enforcement with user-friendly warnings

- **Static vs Dynamic Imports**: Keep it simple
  - Prefer static imports for service dependencies
  - Only use dynamic imports when truly needed (conditional loading)
  - Static imports are more reliable and better for TypeScript
  - Issue discovered when dynamic import caused runtime errors

### Navigation Stability Patterns (2025-10-30)

Key learnings from navigation system debugging and migration:

- **Avoid Unstable APIs in Production**:
  - The "unstable" prefix means it - NativeTabs had race conditions and initialization issues
  - Always prefer stable, battle-tested components for production apps
  - Migration path: `unstable-native-tabs` → stable `Tabs` from expo-router

- **Stack Navigator screenOptions Pitfall**:
  - Using `screenOptions` on Stack creates invisible header overlays even with `headerShown: false`
  - These overlays block touch events on child components (tabs, buttons, etc.)
  - Solution: Always use individual `options` on each Stack.Screen

- **Navigation State Timing**:
  - Immediate redirects (`<Redirect />`) can cause navigation state race conditions
  - Use delayed navigation with `setTimeout` and `router.replace()` for module transitions
  - Small delays (100ms) allow navigation state to stabilize

- **Haptic Feedback Enhancement**:
  - Adding haptic feedback to tab presses significantly improves perceived responsiveness
  - Implementation is simple with `expo-haptics` but has high UX impact
  - Use `ImpactFeedbackStyle.Light` for subtle, pleasant feedback

- **Debugging Navigation Issues**:
  - Systematic elimination approach: Try one fix at a time and document results
  - Check for invisible overlays first (most common cause of unresponsive UI)
  - Test both iOS and Android - navigation behavior can differ
  - Clean builds don't always help - often it's a code issue, not cache

### API Structure

- **Routes**: All routes in `apps/api/src/v1/` with authentication middleware from `@hono/clerk-auth`
- **Services**: Business logic in service files (`*.service.ts`)
- **Validation**: Use Zod schemas from `@nv-internal/validation`
- **Database**: Use Prisma client from `@nv-internal/prisma-client`
- **Testing**: Jest tests in `__tests__/` directories alongside source files
- **Transactions**: Use transactions for multi-model operations
- **Activity Logging**: Log all state changes to Activity table
  - **Unified Event Log**: Activity model serves as single source for all events (check-ins, payments, status changes, comments)
  - **Flexible Queries**: Query by action type (e.g., `TASK_CHECKED_IN`, `TASK_COMMENTED`) for specific events
  - **Feature Pattern**: Use Activity for any feature that logs events (zero new tables needed)
  - **Implementation Example**: Task comments use TASK_COMMENTED action with comment in payload
- **Pagination**: Use cursor-based pagination for lists
- **Authentication**: Optimized to use JWT claims (see [Auth Optimization](./docs/architecture/patterns/auth-optimization.md))

### Mobile App Structure

- **Routing**: Expo Router file-based routing in `apps/mobile/app/`
  - **⚠️ CRITICAL WARNING**: Never use `screenOptions` with Stack navigators wrapping Tabs - it creates invisible overlays that block touch events
  - **✅ CORRECT**: Use individual `options` on each Stack.Screen
  - **❌ WRONG**: `<Stack screenOptions={{ headerShown: false }}>`
  - **Tab Navigation**: Using stable `Tabs` from expo-router (NOT unstable NativeTabs)
  - **Haptic Feedback**: Tab presses trigger light haptic feedback for better UX
  - See [Tabs Navigation Pattern](./docs/architecture/patterns/tabs-navigation.md) for details
- **Authentication**: Clerk SDK with protected routes using auth state
- **API Calls**: Use `callHonoApi` utility for type-safe API calls
- **State Management**: TanStack Query with aggressive caching (1 week gcTime)
  - **Comparison Data**: Fetch multiple periods in parallel for month-over-month comparisons
  - **Smart Invalidation**: Invalidate queries based on parameter changes
- **Styling**: NativeWind (Tailwind for React Native) with sorted classes
  - **className Composition**: Use the `cn` utility from `@/lib/utils` for composing classNames with conditional logic
  - The `cn` utility combines `clsx` and `tailwind-merge` to properly merge and deduplicate Tailwind classes
  - **Change Indicators**: Use color-coded badges (green/red/gray) for metric changes
- **Forms**: Present forms and inputs as modals
- **Components**: Follow existing component structure in `components/` directory
- **Accessibility**: All interactive elements have proper accessibility props (see [Mobile-MCP Testing](./docs/testing/mobile-mcp.md))
- **UX Patterns**:
  - **Pull-to-Refresh**: Implement with haptic feedback on key screens
  - **Bottom Sheets**: Use for selectors (employees, dates) instead of dropdowns
  - **Progressive Disclosure**: Show key metrics first, details on demand

### Database Schema

- **Models**: Task, Customer, GeoLocation, Activity, Attachment, Payment
- **IDs**: Use prefixed IDs for readability (cust*, geo*, act_*, pay_*)
- **Task Status**: PREPARING → READY → IN_PROGRESS → ON_HOLD → COMPLETED
- **Client**: Generated to `packages/prisma-client/generated/`

### Database Optimization Patterns

**Index Strategies** (from Employee Summary implementation):
- **GIN Indexes**: Use for PostgreSQL array columns (e.g., `assigneeIds`)
  ```sql
  CREATE INDEX "Task_assigneeIds_idx" USING GIN ("assigneeIds");
  ```
- **Composite Indexes**: Combine frequently queried columns
  ```sql
  CREATE INDEX "Task_status_completedAt_idx" ON "Task" (status, "completedAt");
  ```
- **Partial Indexes**: Filter at index level for better performance
  ```sql
  CREATE INDEX "Activity_checkins_idx" ON "Activity" ("userId", "createdAt")
  WHERE "action" = 'TASK_CHECKED_IN';
  ```
- **Use CONCURRENTLY**: Avoid locking tables during index creation in production

#### SearchableText Pattern (2025-10-30)

**Optimized Search Implementation**: Pre-computed searchable text fields for performance and simplicity.

**Problem Solved**:
- Complex 7-field OR queries with post-processing (140 lines of code)
- Poor performance with multiple JOINs and in-memory filtering
- Broken pagination due to post-query filtering
- Vietnamese accent normalization happening at query time

**Solution Architecture**:
```typescript
// Build searchable text at write time
function buildSearchableText(record: SomeModel): string {
  const parts = [field1, field2, field3].filter(Boolean)
  return parts
    .map(part => normalizeForSearch(part.trim().replace(/\s+/g, ' ')))
    .join(' ')
}

// Simple query at read time
const results = await prisma.task.findMany({
  where: { searchableText: { contains: normalizedQuery, mode: 'insensitive' } }
})
```

**Benefits**:
- **70% code reduction**: 140 lines → 50 lines
- **2-3x faster queries**: Single indexed field vs multiple JOINs
- **Perfect pagination**: No post-processing needed
- **Type safety**: Full Prisma support maintained

**Implementation Details**: See `.claude/tasks/20251030-094500-implement-searchable-text-field.md`

**When to Use**:
- Multi-field search requirements
- Need for text normalization (accents, case, whitespace)
- Performance-critical search operations
- When pagination accuracy is important

**Anti-pattern to Avoid**:
```typescript
// ❌ Don't do this - complex and slow
const searchConditions = [
  { field1: { contains: query } },
  { field2: { contains: query } },
  { relation: { field3: { contains: query } } },
  // ... many more conditions
]
const results = await prisma.model.findMany({ where: { OR: searchConditions } })
// Then filter results in memory for normalization...
```

## Library Documentation

**IMPORTANT**: Always use the **context7 MCP** to fetch the latest library documentation instead of relying on knowledge cutoff.

### When to Use Context7 MCP

Use context7 MCP tools for:
- ✅ Checking latest API changes and features
- ✅ Verifying correct usage patterns
- ✅ Finding code examples from official docs
- ✅ Ensuring compatibility with current versions
- ✅ Discovering new features or deprecations

### How to Use Context7

1. **Resolve Library ID** (first time for each library):
   ```
   mcp__context7__resolve-library-id
   libraryName: "prisma" (or "next.js", "hono", etc.)
   ```

2. **Get Documentation**:
   ```
   mcp__context7__get-library-docs
   context7CompatibleLibraryID: "/prisma/docs" (from step 1)
   topic: "transactions" (optional, to focus on specific topic)
   tokens: 5000 (optional, default is 5000)
   ```

### Common Libraries in This Project

When working with these libraries, ALWAYS check context7 first:
- **Hono**: Web framework for API routes
- **Prisma**: Database ORM and client
- **React Native**: Mobile app framework
- **Expo**: React Native tooling and SDK
- **TanStack Query**: Data fetching and caching
- **Clerk**: Authentication
- **Zod**: Schema validation
- **date-fns-tz**: Timezone handling

### Example Workflow

```
User: "How do I use Prisma transactions?"
Assistant: Let me check the latest Prisma documentation...
1. Uses mcp__context7__resolve-library-id with "prisma"
2. Uses mcp__context7__get-library-docs with topic "transactions"
3. Provides accurate answer based on latest docs
```

## Development Guidelines

### Code Style

- **TypeScript**: Strict typing throughout
- **Biome**: For formatting and linting (replaces ESLint/Prettier)
- **File Size**: Keep files small and readable
- **Language**: Support Vietnamese language where applicable

### Environment Variables (Mobile)

The mobile app uses **Expo's native environment management pattern** (as of 2025-11-07):

- **Single Variable Names**: Each environment variable has one name (e.g., `EXPO_PUBLIC_API_URL`)
- **Build Profile Management**: Values are defined per build profile in `eas.json`
- **No Runtime Switching**: Environment is determined at build time, not runtime
- **Platform Exception**: Only Google Maps keys remain platform-specific (iOS/Android)

**Pattern**:
```typescript
// Access environment variables directly (no suffixes or fallbacks)
const apiUrl = process.env.EXPO_PUBLIC_API_URL

// Platform-specific exception (Google Maps only)
const googleMapsKey = Platform.select({
  ios: process.env.EXPO_PUBLIC_GOOGLE_MAPS_API_KEY_IOS,
  android: process.env.EXPO_PUBLIC_GOOGLE_MAPS_API_KEY_ANDROID,
})
```

**Build Profiles** (`eas.json`):
- Each profile (`development`, `staging`, `production`) defines its own environment values
- No need for suffixed variables or runtime selection
- See `.claude/tasks/20251107-100000-environment-variable-refactoring.md` for migration details

### Security

- **Authentication**: All API routes require authentication middleware
- **Input Validation**: Server-side validation with Zod schemas
- **Transactions**: Use for multi-model operations
- **No Secrets**: Never hardcode credentials; use environment variables

### Testing

- **API Tests**: Jest tests in `__tests__/` directories
- **Coverage**: Target critical paths
- **Test Environment**: Node environment with ts-jest preset
- **Test Files**: Named `*.test.ts` alongside source files
- **Mock-Based Testing**: All backend tests use mocks exclusively (NEVER real database)
  - **Safety**: Zero database access prevents data corruption
  - **Speed**: 211 tests complete in ~1.36 seconds
  - **Pattern**: Mock `getPrisma()` to intercept all DB calls
  - **Utilities**: Use `createMockPrismaClient()` and `resetPrismaMock()`
  - **Documentation**: See `apps/api/README.md#testing` for patterns

## Development Commands

For complete command reference, see [Development Commands](./docs/development/commands.md).

**Quick reference**:
```bash
# Root commands
pnpm install              # Install dependencies
pnpm test                 # Run all tests
pnpm biome:check          # Format and lint

# API development
npx vc dev                # Start API server (from root)
pnpm --filter @nv-internal/api test  # Run API tests

# Mobile development
pnpm dev                  # Start Expo (from apps/mobile/)
pnpm ios                  # Run on iOS
pnpm android              # Run on Android
```

## Task Documentation

**Documentation Structure**: See `.claude/memory/documentation-structure.md` for complete documentation organization standards.

**IMPORTANT**: When implementing features or fixes, always document your work in `.claude/tasks/`:

1. **Create Task Files**: Use format `YYYYMMDD-HHMMSS-description.md` for descending sort by creation time
   - Date and time in UTC
   - Example: `20251022-224500-fix-attachment-counting-and-ui.md`

2. **Keep Documentation Updated**: Update task files throughout the implementation process
   - Document problem analysis
   - Track implementation steps and progress
   - Record testing results and decisions made
   - Note any deviations from the original plan

3. **Benefits**:
   - Provides audit trail of changes
   - Helps onboard team members
   - Documents architectural decisions
   - Makes it easy to review past implementations

## V1 Feature Planning

**IMPORTANT**: Before implementing v1 features, review and improve the plans in `.claude/plans/v1/`:

### Planning Structure

- **Master Plan**: `.claude/plans/v1/README.md` - Navigation, progress tracking, timeline
- **Feature Plans**: Detailed specifications for each major feature
  - `01-payment-system.md` - Payment tracking & invoices
  - `02-checkin-checkout.md` - GPS-verified check-in/out
  - `03-monthly-reports.md` - Employee performance reports
  - `04-task-crud.md` - Edit/delete tasks
  - `05-employee-management.md` - Profile updates & deletion

### Before Implementation

1. **Review the feature plan** - Read the complete specification
2. **Brainstorm improvements** - Identify edge cases, security issues, performance concerns
3. **Update the plan** - Document improvements before writing code
4. **Create task file** - Link task documentation to the v1 plan
5. **Implement** - Follow the improved plan
6. **Update status** - Mark plan sections as completed

### Linking Plans to Tasks

When creating task files for v1 features:
- Reference the plan file: "Implements Phase 1 from `.claude/plans/v1/01-payment-system.md`"
- Note any deviations from the plan with rationale
- Update plan status when task is completed

## Enhancement Ideas Documentation

**For future features and optimizations not in v1 scope**, document in `.claude/enhancements/`:

### When to Create Enhancement Documentation
- Performance optimizations discovered during development
- UX improvements suggested by users or identified during testing
- Technical debt reduction opportunities
- Feature extensions beyond v1 requirements
- Edge case handling improvements

See `.claude/enhancements/` for detailed specifications and `.claude/enhancements/20251024-120300-enhancement-priorities.md` for priority roadmap.

## Observability Strategy (Partially Implemented) ✅ UPDATED

**Status**: 🚧 Feature flags implemented, error tracking pending

### PostHog Integration

The project uses **PostHog** for comprehensive observability, providing unified analytics, feature flags, and error tracking.

**Implementation Status**:
- ✅ **Feature Flags**: Implemented and working in production (2025-11-07)
  - First flag: `task-list-filter-enabled-admin`
  - Naming convention established: `feature-name-role` format
  - Hook and patterns documented
- ⏳ **Analytics**: SDK installed, basic events ready
- ⏳ **Error Tracking**: Planned, replacing Sentry
- **Cost**: $0/month (free tier: 1M events, project uses ~30k events/month)
- **Risk**: LOW (with rollback plan via feature flag)

**Documentation**:
- Feature Flags Guide: `.claude/docs/feature-flags-guide.md`
- First Flag Implementation: `.claude/tasks/20251107-043354-first-feature-flag-task-list-filter-admin.md`
- Full Implementation Plan: `.claude/enhancements/20251031-posthog-observability-implementation.md`
- Error Tracking Guide: `.claude/docs/error-tracking-guide.md`

**Key Implementation Patterns**:

#### Mobile Patterns
```typescript
// Async initialization (CRITICAL)
await posthog.initAsync();

// Error queue for reliability
const errorQueue = [];

// Manual screen tracking
posthog.screen(semanticName);

// Feature flags with PostHogProvider
<PostHogProvider client={posthog}>
  {children}
</PostHogProvider>
```

#### API Patterns
```typescript
// Root error handler (NOT middleware)
app.onError(async (err, c) => {
  await captureAndFlush('$exception', {...});
});

// Serverless-safe flushing
await posthog.captureImmediate({...});

// Request sanitization
sanitizeRequestBody(body);

// Stack trace scrubbing
scrubStackTrace(stack);
```

**Key Benefits**:
- ✅ Works with Expo Go (no dev builds needed!)
- Data-driven feature rollouts with cached flags
- Complete error tracking with context
- Performance monitoring built-in
- Offline support with event queuing
- Zero ongoing costs for our scale

## Backend Architecture Refactoring (Planned)

**Status**: 📋 Planning complete, awaiting team approval

A comprehensive backend refactoring plan has been created to improve code quality, maintainability, and align with SOLID principles while optimizing for Hono + Vercel serverless environments.

### Documentation

- **Full Plan**: `.claude/tasks/20251024-212000-backend-refactoring-plan-solid.md`
- **Executive Summary**: `.claude/tasks/REFACTORING-EXECUTIVE-SUMMARY.md`
- **Timeline**: 18-20 weeks (Week 0 + Phases 0-6)

### When to Apply

**Not yet implemented** - This is a planned architectural improvement. Current code should follow existing patterns documented in this file and `docs/architecture/patterns/`. Post-refactoring, new patterns will replace current service layer implementation.

## Important File Locations

- **API Routes**: `apps/api/src/v1/*/` (route.ts and service.ts files)
- **Mobile Screens**: `apps/mobile/app/` (file-based routing)
- **Database Schema**: `apps/api/prisma/schema.prisma`
- **Shared Validation**: `packages/validation/`
- **Shared Prisma Client**: `packages/prisma-client/`
- **Architecture Patterns**: `docs/architecture/patterns/` (detailed implementation patterns)
- **Development Guides**: `docs/development/` (setup, commands, workflows)
- **Testing Guides**: `docs/testing/` (testing strategies)
- **QA Documentation**: `.claude/qa/` (test plans, scenarios, and results)
  - Test Plans: `.claude/qa/test-plans/` (feature test specifications)
  - Test Scenarios: `.claude/qa/test-scenarios/` (detailed test cases)
  - Test Results: `.claude/qa/test-results/` (execution results and bugs)
- **Feature Flags**: `.claude/docs/feature-flags-guide.md` (PostHog feature flag patterns and conventions)
- **Task Documentation**: `.claude/tasks/` (implementation tracking)
- **V1 Feature Plans**: `.claude/plans/v1/` (detailed feature specifications & roadmap)
- **Enhancement Ideas**: `.claude/enhancements/` (future features & optimizations)
- **Documentation Standards**: `.claude/memory/` (project-wide conventions and patterns)
- **Refactoring Plans**: `.claude/tasks/REFACTORING-*.md` (architecture refactoring documentation)

## Project Scale Context

- **Team Size**: Small business (< 50 users)
- **Industry**: Air conditioning services in Vietnam
- **Platform**: Mobile-first for field workers
- **Features**: Task management, check-in/check-out, location tracking, photo attachments

## Environment Setup

For complete setup guide, see [Environment Setup](./docs/development/setup.md).

**Quick start**:
1. Install pnpm package manager
2. Clone repository and run `pnpm install`
3. Set up environment variables (see .env.example files)
4. For local database: Start Docker Compose in `apps/api/`
5. Start API with `npx vc dev` from root
6. Start mobile app with `pnpm dev` in `apps/mobile/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duongdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/duongdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
