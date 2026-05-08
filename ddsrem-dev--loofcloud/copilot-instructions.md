## loofcloud

> This file provides guidance to AI coding agents when working with code in this repository.

This file provides guidance to AI coding agents when working with code in this repository.

# LoofCloud Project Conventions

## 1. Commit Conventions

Use **Conventional Commits** for consistent history and tooling.

### Format

```
<type>(<scope>): <subject>

[optional body]
[optional footer]

Co-authored-by: <AI Name> <ai-email@example.com>
```

- **type** (required): Commit type; see table below.
- **scope** (required): Affected area, e.g. `app`, `api`, `components`, `contexts`, `hooks`, `config`, `deps`, `docs`.
- **subject** (required): Short description, ~50 chars; no period at the end.
- **Co-authored-by** (required when AI is involved): Footer attributing the AI assistant; use the agent’s canonical name and email (e.g. `Cursor Agent <cursoragent@cursor.com>`).

### Type Reference

| type     | Description                          |
|----------|--------------------------------------|
| feat     | New feature                          |
| fix      | Bug fix                              |
| docs     | Documentation only (README, comments)|
| style    | Code style (no logic change)         |
| refactor | Refactor (not new feature/fix)       |
| perf     | Performance improvement              |
| test     | Tests                                |
| chore    | Build/tooling/deps, etc.             |

### Examples

```
feat(components): add PWA install banner for web

Co-authored-by: Cursor Agent <cursoragent@cursor.com>
```

```
fix(api): correct auth header for token refresh

Co-authored-by: Cursor Agent <cursoragent@cursor.com>
```

### Rules

- Commit messages must be in **English**.
- **AI Co-author required**: Commits made by or with assistance from an AI agent **must** include a `Co-authored-by:` line in the footer.
- One logical change per commit; split unrelated changes into separate commits.
- For breaking changes, describe in body or footer; use `BREAKING CHANGE:` when needed.

---

## 2. TypeScript / React Conventions

### 1. Style and Format

- **TypeScript**: Strict mode enabled; avoid `any` where possible; use explicit return types for exported functions.
- **Formatting**: 2-space indent; trailing commas in multi-line structures for cleaner diffs.
- **Strings**: Prefer double quotes `"`; use single quotes when embedding double quotes.
- **JSX**: Use double quotes for attribute values; self-close tags when no children.

### 2. Path Aliases

- Use the `@/` alias for project imports (maps to repo root).
- Prefer `@/api`, `@/components/...`, `@/contexts/...`, `@/constants/...`, `@/hooks/...` over relative paths like `../../api`.

```ts
import { apiLogin } from '@/api'
import { useAuth } from '@/contexts/AuthContext'
import { StyledInput } from '@/components/shared/StyledInput'
```

### 3. Types and Exports

- Export types with `export type { ... }` or `import type { ... }` where applicable.
- Public API (e.g. under `api/`) and shared components should have clear types; avoid inline object types for props when reused.
- Use `interface` for object shapes and `type` for unions/intersections as appropriate.

### 4. Naming

- **Files**: PascalCase for components (`P115LoginCard.tsx`), camelCase for utilities/hooks (`usePWA.ts`, `auth.ts`).
- **Components**: PascalCase.
- **Functions, variables, hooks**: camelCase.
- **Constants**: UPPER_SNAKE_CASE for true constants; design tokens in `constants/` may use camelCase (e.g. DesignTokens).
- **Private / internal**: not strictly enforced; prefer clear naming over leading underscores in TS/React.

### 5. React and Expo

- **Expo Router**: Routes live under `app/`; use file-based routing; use `router` from `expo-router` for navigation.
- **Tamagui**: Use Tamagui primitives (e.g. `YStack`, `XStack`, `Button`, `Text`) from `tamagui`; avoid passing `style` to Tamagui components when the type does not support it—use Tamagui props (e.g. `backgroundColor`, `padding`) or wrap in a `View` with `style` if needed.
- **Lucide icons**: Pass hex color strings (e.g. `#5bcffa`), not Tamagui tokens like `$primary`, for icon `color` props.
- **Hooks**: Keep hooks pure and side-effectful logic in `useEffect` or event handlers; auth state via `AuthContext`, theme via `ThemeContext`.

### 6. API Layer

- API client and endpoints live under `api/`; entry point is `api/index.ts`.
- Use `authFetch` from `api/client` for authenticated requests; keep request/response types in the same module or in `api/index.ts` re-exports.
- Do not hardcode secrets or tokens; use `EXPO_PUBLIC_*` or env for config (see `.env.example`).

### 7. Error Handling and Logging

- Avoid empty `catch` blocks; handle errors and optionally show user-facing toasts (e.g. `useToast`).
- For API errors, surface a clear message to the user where appropriate; avoid exposing internal stack traces.

### 8. Imports Order

1. React / React Native / Expo (grouped)
2. Blank line
3. Third-party (e.g. tamagui, lucide-react-native)
4. Blank line
5. Project imports (`@/api`, `@/components/...`, `@/contexts/...`, etc.)

---

## 3. Quick Checklist

**Before committing:**

- [ ] Message follows `type(scope): subject` format
- [ ] Footer includes `Co-authored-by: <AI Name> <email>` when AI contributed
- [ ] New or changed public API and shared components have appropriate types
- [ ] Imports use `@/` alias where relevant; no unused imports
- [ ] No hardcoded secrets; env/config as in `.env.example`

**During review:**

- [ ] Logic is correct; edge cases and error paths considered
- [ ] Tamagui usage avoids unsupported `style` prop where it causes type errors
- [ ] Consistent with existing app structure (app/, api/, components/, contexts/)

---
> Source: [DDSRem-Dev/LoofCloud](https://github.com/DDSRem-Dev/LoofCloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
