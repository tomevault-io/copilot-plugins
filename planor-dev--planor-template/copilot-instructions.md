## planor-template

> These rules apply to **every version** and **every task** for this project. They describe the default stack, how the codebase is structured, and the expectations for quality and documentation.


# Project Rules

These rules apply to **every version** and **every task** for this project. They describe the default stack, how the codebase is structured, and the expectations for quality and documentation.

## Stack Baseline

- **Platform**: Expo React Native targeting iOS and Android (Expo SDK 52 or the latest compatible version).
- **Navigation**: Expo Router with file-based routes under the `app/` folder.
- **UI Kit**: React Native Reusables with Tailwind (NativeWind) classes for styling.
- **State & Validation**: Zustand for client-side state, Zod for runtime validation and type inference.
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Edge Functions) **hosted on Supabase Cloud**. All data access flows through Supabase client helpers in `lib/supabase.ts`; never spin up self-hosted Supabase locally.
- **Analytics & Growth**: PostHog analytics initialized in `lib/posthog.ts` and Superwall paywall orchestration in `lib/superwall.ts`. Reuse these clients instead of rolling your own instrumentation.
- **Internationalization**: i18n-js with expo-localization plus typed helpers from `lib/i18n.ts`. All UI strings live in the translation dictionaries.
- **Expo Packages Requirement**: **Always use official Expo packages** for native functionality. For authentication flows use `expo-auth-session` and `expo-web-browser`, for secure storage use `expo-secure-store`, for camera use `expo-camera`, for file system use `expo-file-system`, etc. Never install third-party alternatives (e.g., react-native-community packages) when an `expo-*` equivalent exists. This ensures compatibility, automatic config plugin support, and seamless updates across Expo SDK versions.
- **Authentication Setup**: When implementing OAuth (Google, Apple, etc.), always:
  - Install and configure both `expo-auth-session` and `expo-web-browser` packages
  - Use `expo-web-browser` for opening OAuth flows (never rely on default browser behavior)
  - Configure redirect URLs correctly using the app's bundle ID (iOS) and package name (Android)
  - Set up proper URL schemes in `app.json` matching the OAuth provider requirements
  - Test OAuth flows on both platforms with correct bundle ID/package name configuration
  - Document all required OAuth provider setup steps (Google Cloud Console, Apple Developer Portal, etc.)

## Template & Repository Baseline

- Every repository is bootstrapped **before the first task runs** with `npx create-planor-app@latest <project-slug>`. The CLI clones `Planor-Dev/planor-template` (v1.0.0, CLI v1.0.1) so never rip out or replace that scaffolding.
- Expect the scaffolded structure: `app/` routes, shared helpers under `lib/`, generated Supabase types in `lib/database.types.ts`, analytics/paywall wiring, and the Jest/Testing Library setup. Extend those files instead of recreating them.

## Architecture & Project Structure

- Shared code lives in `components/`, `hooks/`, `lib/`, `stores/`, `services/`, and `utils/`. Reusable UI components belong in `components/ui/`; screen- or flow-specific pieces can live directly under `components/`.
- Do not force a feature-based folder structure; keep routes scoped under `app/` and extend the existing shared directories instead of introducing new `features/` wrappers.
- Use snake_case for file and folder names unless an existing path already defines a different convention you must follow.
- Keep screens thin: orchestration belongs in hooks/services, UI logic in components.
- Never bypass centralized services (e.g., Supabase helpers, shared stores) when touching data or auth.
- Before starting any frontend feature, scaffold localization infrastructure (i18n-js initialization, Expo Localization wiring, default dictionaries, helper hooks) so every UI contribution consumes translations from day one.

## Strict Localization

- Absolutely no hard-coded copy. Every label, placeholder, toast, error, tooltip, and notification must reference a translation key.
- Store translations under `i18n/en.json` (and other locale files once added). Use descriptive keys (e.g., `tasks.aftermathDialogTitle`) so intent is obvious.
- Initialize 'i18n-js' with Expo Localization and fall back to English gracefully. All new strings belong in the dictionary before you render UI.
- When you patch or add features, update the translation files in the same commit so downstream assistants inherit a complete dictionary.

## Documentation Expectations

- Every code file needs a short header comment or docblock capturing **purpose** and **constraints**. Readers should know why the file exists without scanning its body.
- Public modules (components, hooks, utilities, services) must expose JSDoc annotations that explain params, return types, and side effects.
- Use inline comments sparingly to clarify architectural decisions, third-party assumptions, or tricky logic—never to narrate obvious TypeScript.
- Capture README/CHANGELOG or broader documentation needs in each task’s aftermath; apply those updates during the version-end QA/docs task so the docs reflect the fully tested version.

## Coding & Tooling Standards

- TypeScript everywhere. No `.js` files.
- Honor ESLint + Prettier defaults shipped with the repo. Run formatters before submitting changes.
- Use functional React components and hooks. Avoid legacy lifecycle APIs.
- Keep files focused: extract helpers instead of burying large blocks inline.
- Treat error states explicitly—surface actionable messages and log unexpected failures.

## State, Data, and Backend Access

- Zustand stores live in `stores/`. Co-locate slices with related services if they are version-specific.
- Validate all external data (API payloads, form input) with Zod schemas.
- All network/storage/auth logic must go through Supabase service modules inside `lib/` or `services/`. Do not create ad-hoc fetch calls or bypass the generated helpers in `lib/supabase.ts`.
- Supabase runs remotely: always link to the cloud project via `npm run supabase:link` (or `npx supabase link`) before using any Supabase CLI commands. Do not run local/self-hosted Supabase; use the provided `npm run supabase:*` scripts to pull/push schemas and regenerate types.
- Follow the CLI workflow for Supabase (`npm run supabase:link`, `npm run supabase:pull`, `npm run supabase:gen-types`, `npm run supabase:push`) whenever schemas change so `lib/database.types.ts` stays in sync.
- Respect optimistic UI patterns only when stores can roll back gracefully.

## MCP Tools

- **Supabase MCP** is configured and available in your IDE. Use it for all Supabase operations:
  - Run `list_tables`, `get_table_schema` to explore the current database structure before making changes.
  - Use `execute_sql` for ad-hoc queries or debugging.
  - Leverage `apply_migration` to create and apply schema migrations safely.
  - Call `generate_typescript_types` to regenerate `lib/database.types.ts` after schema changes.
- **Prefer MCP over manual CLI** when available—it provides direct feedback in your IDE context and reduces context-switching.
- Always verify schema changes with `get_table_schema` after applying migrations to confirm the expected structure.

## Mobile UI & Design Standards

### Design Philosophy

- **Think Product Designer**: Every screen should feel intentionally designed, not scaffolded. Consider user flows, information hierarchy, and emotional impact before writing code.
- **Mobile-First Mindset**: Design for thumb zones, touch targets (minimum 44×44px), and one-handed use. Primary actions belong in the bottom third of the screen.
- **Avoid AI Aesthetics**: No generic purple gradients, default system fonts, or cookie-cutter layouts. Choose contextual, distinctive design decisions that match the app's purpose and audience.

### Visual Design

- **Typography**: Use meaningful font weight hierarchies (max 3 weights). Body text minimum 16px to prevent zoom. Line height 1.4-1.6 for readability. Leverage NativeWind's font utilities but customize scale when needed.
- **Color System**: Use the CSS-var-backed Tailwind tokens in `tailwind.config.js`: `background/foreground`, `primary|secondary|destructive|muted|accent|popover|card` (each with `DEFAULT` + `foreground` variants), plus `border`, `input`, and `ring`. Keep contrast at 4.5:1+ in both light and dark modes via `dark:` variants.
- **Spacing**: Follow consistent 4px or 8px spacing system. Use generous padding/margins—cramped UI feels unpolished.
- **Visual Hierarchy**: Use size, weight, color, and spacing to guide attention. The most important element should be unmistakably obvious.

### Interaction Design

- **Touch Targets**: All interactive elements ≥44×44px with ≥8px spacing between them.
- **Micro-interactions**: Provide immediate feedback (<100ms) for every tap. Use scale, opacity, or color changes to acknowledge interaction.
- **Gestures**: Implement swipe-to-delete, pull-to-refresh, and bottom sheets where appropriate. Avoid gesture conflicts (e.g., horizontal swipe on a screen with swipeable cards).
- **Loading States**: Use skeleton screens or progress indicators, never blank screens. Show optimistic updates when safe.
- **Error States**: Display actionable error messages with recovery paths. Never show raw error codes to users.

### Animation & Motion

- **Purpose-Driven**: Animate to provide feedback, guide attention, or establish relationships—not for decoration.
- **Performance**: Use transform and opacity properties (GPU-accelerated). Avoid animating layout properties. Target 60fps consistently.
- **Duration**: 200-300ms for most transitions. Faster (100-150ms) for immediate feedback, slower (400-500ms) for page transitions.
- **Easing**: Use natural curves (ease-out for entrances, ease-in for exits). Avoid linear easing except for looping animations.

### Component Standards

- **Consistency**: Reuse components from `components/ui/`. Extend rather than duplicate.
- **Variants**: Support size variants (sm, md, lg) and states (default, hover/pressed, disabled, loading) for all interactive components.
- **Accessibility**: Include proper labels, roles, and focus indicators. Test with screen readers.
- **Empty States**: Design engaging empty states with clear calls-to-action, not just "No data" text.

### Screen Composition

- **Safe Areas**: Respect iOS notches and Android system bars using `useSafeAreaInsets()`.
- **Navigation**: Bottom tabs for 3-5 top-level sections. Use header for context and actions. Keep navigation predictable. If the project ships a bottom nav bar, keep it persistent, safe-area aware, and clearly highlight the active tab.
- **Scrolling**: Make scrollable areas obvious. Use scroll indicators and subtle shadows to show more content exists.
- **Modals & Sheets**: Use bottom sheets for contextual actions, full modals for focus-required tasks. Always provide clear dismiss affordances.

### Quality Checklist

Before considering any screen complete:

- Touch targets meet minimum size and spacing
- Loading, error, and empty states are designed
- Dark mode works without breaking contrast or hierarchy
- Animations run at 60fps on mid-range devices
- Screen reader announces content in logical order
- Design feels cohesive with the rest of the app

### Anti-Patterns to Avoid

- Default system fonts (use distinctive typeface choices)
- Centered text blocks >2 lines (left-align for readability)
- Tiny tap targets or crowded interactive elements
- Sudden content shifts or layout thrashing
- Generic iconography when custom illustrations would elevate experience
- Ignoring platform conventions (e.g., Android back button, iOS swipe-back)

**Remember**: A polished mobile app feels fast, looks intentional, and anticipates user needs. Invest time in micro-interactions, smooth transitions, and thoughtful visual design—these details distinguish professional apps from prototypes.

## Testing & Quality

- The template ships with Jest + `jest-expo` and `@testing-library/react-native` (see `components/__tests__/ui/example-test.tsx`). The dedicated \*\*version-end

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Planor-Dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
