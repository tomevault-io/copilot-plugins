## unoff-template-figma

> Core project rules for {{ pluginName }} Figma plugin development


# Windsurf Rules for {{ pluginName }}

## 📚 Complete Documentation

**For comprehensive patterns, code examples, and best practices, always refer to:**

**[Architecture & Skills Documentation](skills/README.md)**

**Quick links**:
- **Canvas**: [Figma API](skills/canvas/figma-api.md) • [Data Storage](skills/canvas/data-storage.md)
- **Bridge**: [Communication Pattern](skills/bridge/communication-pattern.md) • [Bridge Functions](skills/bridge/bridge-functions.md)
- **Config**: [Global Config](skills/config/global-config.md) • [Feature Flags](skills/config/feature-flags.md) • [Credits System](skills/config/credits-system.md) • [Vite Build](skills/config/vite-build.md) • [Code Quality](skills/config/code-quality.md)
- **UI**: [Component Library](skills/ui/component-library.md) • [Component Patterns](skills/ui/component-patterns.md) • [External Services](skills/ui/external-services.md) • [State Management](skills/ui/state-management.md) • [i18n](skills/ui/i18n.md) • [Types System](skills/ui/types-system.md) • [Error Handling](skills/ui/error-handling.md) • [CSS & Theming](skills/ui/css-theming.md) • [Accessibility](skills/ui/accessibility.md) • [Performance](skills/ui/performance.md) • [App Bootstrap](skills/ui/app-bootstrap.md)
- **Externals**: [Implement Design](skills/externals/implement-design) — Figma spec document → code workflow (annotations, MCP, unoff-ui) • [Payment Systems](skills/externals/payment-systems.md) — Figma built-in vs Lemon Squeezy, must choose one

---

## Project Overview
Figma plugin built with TypeScript, Preact (aliased via preact/compat), and Vite. PureComponent class components + HOCs. Nanostores for lightweight state. Separates Canvas logic (Figma API) from UI logic (Preact).

## Core Architecture

### Two-Context System
1. **Main Thread (Bridges)**: Figma API, no DOM
2. **UI Thread (React)**: React UI, no Figma API
3. **Communication**: PostMessage API

### Directory Structure

#### `/src/bridges/` - Figma Canvas
**Key File**: `loadUI.ts` - Message router & communication hub

**What to do**: Figma API operations (nodes, styles, variables, storage)
**What NOT to do**: React, DOM, state management

**Subdirectories**: `/checks/`, `/plans/`

#### `/src/app/` - React UI
**Key Directories**:
- `ui/` - React components
- `stores/` - Nanostores atoms
- `external/` - Services (auth, tracking, monitoring)
- `types/` - TypeScript definitions
- `config/` - Contexts (Config, Theme)
- `content/` - Assets, i18n
- `utils/` - UI utilities
  - **`pluginMessage.ts`** - ⭐ UI → Canvas messages

#### `/src/utils/` - Global Utilities
Shared utilities (`i18n.ts`, `setData.ts`)

## Communication Pattern

```typescript
// UI → Canvas
import { sendPluginMessage } from '../utils/pluginMessage'
sendPluginMessage({ pluginMessage: { type: 'ACTION', data: {...} } })

// Canvas → UI (in loadUI.ts)
figma.ui.onmessage = async (msg) => {
  const actions = {
    ACTION: async () => {
      // Figma API operations
      figma.ui.postMessage({ type: 'RESULT', data: {...} })
    }
  }
  if (actions[msg.type]) await actions[msg.type]()
}
```

## Critical Rules

### ✅ DO
- Separate UI and Canvas logic completely
- Use `sendPluginMessage()` for UI → Canvas
- Use `@unoff/ui` components
- Use `FeatureStatus` for permissions
- Define TypeScript types for messages
- Handle errors in bridge functions
- **Refer to skills docs for detailed patterns**

### ❌ DON'T
- Mix Figma API in React components
- Mix React in bridge files
- Use `parent.postMessage()` directly
- Create UI components when unoff-ui has them
- Use `any` type
- Skip error handling

## Component Library

```typescript
// Standard imports
import { doClassnames, FeatureStatus } from '@unoff/utils'
import { Bar, Button, Dropdown, Icon, Input, Menu, layouts, texts } from '@unoff/ui'

// FeatureStatus pattern
static features = (planStatus, config, service, editor) => ({
  FEATURE: new FeatureStatus({
    features: config.features,
    featureName: 'FEATURE',
    planStatus, currentService: service, currentEditor: editor
  })
})
```

See [Component Library](skills/ui/component-library.md) for complete examples.

## Tech Stack
- TypeScript (strict mode) + Preact (aliased via preact/compat) + Vite
- PureComponent class components + HOCs (WithConfig, WithTranslation)
- Nanostores (lightweight atoms, $prefix convention)
- @unoff/ui (component library) + @unoff/utils (utilities)
- Tolgee (UI translations) + createI18n (Canvas translations)
{{#isSupabaseEnabled}}
- Supabase (auth & database)
{{/isSupabaseEnabled}}
{{#isSentryEnabled}}
- Sentry (error monitoring)
{{/isSentryEnabled}}
{{#isMixpanelEnabled}}
- Mixpanel (analytics)
{{/isMixpanelEnabled}}

## Message Naming
- UI → Canvas: `VERB_NOUN` (e.g., `CREATE_RECTANGLE`)
- Canvas → UI: `NOUN_PAST_TENSE` (e.g., `RECTANGLE_CREATED`)

## TypeScript Standards
- Strict mode always
- No `any` types
- Use `interface` for objects, `type` for unions
- Define message types

## Preact Patterns
- PureComponent class components (NOT functional)
- HOCs: WithConfig (class-based) + WithTranslation (functional wrapper)
- `forceUpdate()` for language changes
- Nanostores atoms with `$prefix` convention

## Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `ColorPicker.tsx` |
| Functions | camelCase | `getUserData()` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Types | PascalCase | `UserPreferences` |

## When Generating Code

1. **Check skills documentation** for patterns
2. Follow existing codebase patterns
3. Use TypeScript strict mode
4. Add error handling
5. Use unoff-ui components
6. Add JSDoc for public APIs

## Environment Variables
{{#isSupabaseEnabled}}
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_PUBLIC_ANON_KEY`
{{/isSupabaseEnabled}}
{{#isSentryEnabled}}
- `VITE_SENTRY_DSN`
{{/isSentryEnabled}}
{{#isMixpanelEnabled}}
- `VITE_MIXPANEL_TOKEN`
{{/isMixpanelEnabled}}

## Performance
- `PureComponent` for shallow comparison render optimization
- Remove DOM for inactive features (not `display: none`)
- Conditional service init (Sentry/Mixpanel only in production)
- Build: `viteSingleFile` (zero network requests), platform CSS stripping
- Batch Figma operations
- See [Performance Guide](skills/ui/performance.md)

## Additional Resources
- [Architecture Documentation](ARCHITECTURE.md)
- [Skills Documentation](skills/README.md)

---

**Remember**: Two-context architecture. Keep Canvas and UI separate. Communicate through messages. Refer to skills docs for detailed patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yelbolt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
