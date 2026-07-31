## odh-tec

> > **Note for AI Assistants**: This is a frontend-specific context file for the ODH-TEC React application. For project overview, see root [CLAUDE.md](../CLAUDE.md). For backend context, see [backend/CLAUDE.md](../backend/CLAUDE.md).

# CLAUDE.md - ODH-TEC Frontend Context

> **Note for AI Assistants**: This is a frontend-specific context file for the ODH-TEC React application. For project overview, see root [CLAUDE.md](../CLAUDE.md). For backend context, see [backend/CLAUDE.md](../backend/CLAUDE.md).

## 🎯 Frontend Overview

**odh-tec-frontend** - React 18 application with TypeScript, Webpack, and PatternFly 6 component library.

**Technology Stack**: React 18, PatternFly 6, React Router v7, TypeScript, Webpack
**Development**: Port 9000 with Webpack HMR (Hot Module Replacement)
**Production**: Built and served statically by Fastify backend on port 8888

**For detailed architecture**, see [Frontend Architecture](../docs/architecture/frontend-architecture.md).

## 🎨 PatternFly 6 Critical Requirements

⚠️ **MANDATORY**: Follow the [PatternFly 6 Development Guide](../docs/development/pf6-guide/README.md) as the **AUTHORITATIVE SOURCE** for all UI development.

### Essential Rules

1. **Class Prefix**: ALL PatternFly classes MUST use `pf-v6-` prefix
2. **Design Tokens**: Use semantic tokens only, never hardcode colors
3. **Component Import**: Import from `@patternfly/react-core` v6 and other @patternfly libraries
4. **Theme Testing**: Test in both light and dark themes
5. **Table Patterns**: Follow guide's table implementation (current code may be outdated)

### Common Mistakes and Token Usage

**Critical rules** - See [`docs/development/pf6-guide/guidelines/styling-standards.md`](../docs/development/pf6-guide/guidelines/styling-standards.md) for complete guide:

- ✅ **ALWAYS** use `pf-v6-` prefix for component classes
- ✅ **ALWAYS** use `--pf-t--` prefix for design tokens (semantic tokens with `-t-`)
- ✅ Choose tokens by **meaning** (e.g., `--pf-t--global--color--brand--default`), not appearance
- ❌ **NEVER** hardcode colors or measurements
- ❌ **NEVER** use legacy `--pf-v6-global--` tokens or numbered base tokens

### Component Import Pattern

```tsx
import { Button, Card, Page, PageSection } from '@patternfly/react-core';
import { Table, Thead, Tbody, Tr, Th, Td } from '@patternfly/react-table';
import { TrashIcon, UploadIcon } from '@patternfly/react-icons';
```

**Version**: PatternFly 6.2.x (NOT PatternFly 5)

## 🗃️ State Management Philosophy

- **Local State First**: Use `useState` for component-specific state
- **Context for Global**: UserContext for shared state
- **EventEmitter for Cross-Component**: Use emitter for decoupled communication (upload progress, notifications)
- **No Redux Active**: Redux folder exists but not actively used
- **No React Query**: Direct API calls with axios and local loading/error states

## 🎯 Component Development Checklist

### Before Creating ANY Component

1. **Search for similar components first** - Use `find_symbol` and `search_for_pattern`
2. **Follow PatternFly 6 requirements** - ALWAYS use `pf-v6-` prefix, semantic tokens, v6 imports
3. **Use established patterns** - Check existing components (StorageBrowser, Buckets, Settings)

### Critical Rules for ALL Components

1. **Error Handling**: MUST use `Emitter.emit('notification', { variant, title, description })` for user-facing errors
   - Use `.catch()` with axios calls
   - Log errors with `console.error()` for debugging
   - Display user-friendly notifications via EventEmitter

2. **Data Fetching**: Use direct axios calls with local state
   - Set loading state before call
   - Handle errors in `.catch()`
   - Update component state on success

3. **Internationalization**: MUST use `t()` function - never hardcode user-facing text
   - Import from `react-i18next`
   - Wrap all strings in `t('key')`

4. **Accessibility**: MUST include ARIA labels and keyboard navigation
   - Add `aria-label` to interactive elements
   - Ensure keyboard navigation works
   - Test with screen readers when possible

5. **PatternFly 6**: MUST use `pf-v6-` prefix and semantic design tokens
   - Never hardcode colors or spacing
   - Use `--pf-t--` tokens for styling
   - Test in both light and dark themes

### Component Pattern Example

```tsx
const MyComponent: React.FC = () => {
  const [loading, setLoading] = React.useState(false);
  const [data, setData] = React.useState(null);
  const { t } = useTranslation();

  const fetchData = () => {
    setLoading(true);
    axios.get('/api/endpoint')
      .then(response => {
        setData(response.data);
        setLoading(false);
      })
      .catch(error => {
        console.error('Error fetching data:', error);
        Emitter.emit('notification', {
          variant: 'warning',
          title: t('error.fetch.title'),
          description: error.response?.data?.message || t('error.fetch.default')
        });
        setLoading(false);
      });
  };

  return <div>{/* Component JSX */}</div>;
};
```

## 🚀 Essential Development Commands

```bash
# Development server with HMR
npm run start:dev

# Building
npm run build          # TypeScript check + clean + webpack production build

# Testing
npm run test           # Run Jest tests
npm run test:coverage  # Coverage report

# Code quality
npm run lint           # ESLint check
npm run type-check     # TypeScript type checking
npm run format         # Prettier format

# CI pipeline
npm run ci-checks      # type-check + lint + test:coverage
```

**For complete workflow**, see [Development Workflow](../docs/development/development-workflow.md).

## 📁 Component Organization

Main components in `src/app/components/`:
- **AppLayout** - Main layout with navigation sidebar
- **StorageBrowser** - Unified storage browser for S3 and local storage with upload/download
- **Buckets** - S3 bucket management
- **VramEstimator** - GPU VRAM calculation tool
- **Settings** - S3 connection configuration
- **UserContext** - Simple user state management
- **DocumentRenderer** - Markdown/document viewer
- **NotFound** - 404 page

**For component patterns and examples**, see [Frontend Architecture](../docs/architecture/frontend-architecture.md).

## 🌐 Routing

- React Router v7 for navigation
- Routes defined in `src/app/routes.tsx`
- Main routes: `/browse/:locationId?/:path?` (Storage Browser), `/buckets`, `/gpu/vram-estimator`, `/settings`
- No protected routes (no authentication)

### URL Encoding Strategy

The application uses **intentionally different encoding strategies** for URL parameters:

**LocationId (NOT encoded)**:
- S3 bucket names: Validated to URL-safe `[a-z0-9-]` (see `backend/src/utils/validation.ts`)
- PVC locations: Use pattern `local-0`, `local-1` (always URL-safe)
- Benefit: Human-readable URLs like `/browse/my-bucket`

**Path (Base64-encoded)**:
- Contains slashes, spaces, special characters
- Example: `models/llama/config.json` → `bW9kZWxzL2xsYW1hL2NvbmZpZy5qc29u`
- Benefit: Handles all characters without URL encoding issues

**For full details**, see [Frontend Architecture - URL Encoding Strategy](../docs/architecture/frontend-architecture.md#url-encoding-strategy).

## 🔌 API Integration

**Backend API**: Fastify server on port 8888

**API Pattern**: Direct axios calls in components (no service layer)

```tsx
const response = await axios.get('/api/buckets');
```

**Main Endpoints**:
- `/api/buckets` - Bucket operations
- `/api/objects` - Object operations (upload, download, list, delete)
- `/api/settings` - Connection settings
- `/api/disclaimer` - Application info

**For API details**, see [Backend Architecture](../docs/architecture/backend-architecture.md).

## 🧪 Testing

**Framework**: Jest with React Testing Library

**Key Patterns**:
- Component testing with `@testing-library/react`
- User interactions with `@testing-library/user-event`
- Mock axios for API calls
- Use `waitFor` for async operations

### PatternFly 6 Testing Patterns

- **Modals**: Use `role="dialog"` to query modals
- **Dropdowns**: Use `role="menuitem"` for dropdown options
- **Buttons**: Use `getByRole('button', { name: 'Button Text' })`
- **Forms**: Query by `role="textbox"`, `role="combobox"`, etc.

**Example Test**:
```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('should open modal on button click', async () => {
  const user = userEvent.setup();
  render(<MyComponent />);

  const button = screen.getByRole('button', { name: 'Open Modal' });
  await user.click(button);

  expect(screen.getByRole('dialog')).toBeInTheDocument();
});
```

**For testing patterns**, see [PatternFly 6 Testing Patterns](../docs/development/pf6-guide/testing-patterns/README.md).

## 🎨 Styling Guidelines

- Use PatternFly 6 design tokens exclusively
- Support dark theme with semantic tokens (they auto-adapt)
- Avoid hardcoded values - use `--pf-t--` tokens
- Test in both light and dark themes

**Example**:
```css
/* ✅ CORRECT - Semantic token */
.my-element {
  color: var(--pf-t--global--color--brand--default);
  padding: var(--pf-t--global--spacer--md);
}

/* ❌ WRONG - Hardcoded value */
.my-element {
  color: #0066cc;
  padding: 16px;
}
```

## 📚 Project Structure

```
frontend/
├── src/
│   ├── app/
│   │   ├── components/      # React components
│   │   ├── utils/           # Utilities (EventEmitter, hooks)
│   │   ├── assets/          # Images and icons
│   │   ├── routes.tsx       # Route definitions
│   │   ├── config.tsx       # App configuration
│   │   └── index.tsx        # App component
│   ├── i18n/                # Internationalization
│   ├── redux/               # Redux (not actively used)
│   └── index.tsx            # Entry point
├── dist/                    # Webpack build output
├── webpack.common.js        # Base config
├── webpack.dev.js           # Development config
└── webpack.prod.js          # Production config
```

## ⚠️ Key Implementation Guidelines

### For AI Assistants

**DO:**
- ✅ Use PatternFly 6 components with `pf-v6-` prefix
- ✅ Import from `@patternfly/react-core`, `@patternfly/react-table`, `@patternfly/react-icons` v6
- ✅ Use `--pf-t--` semantic design tokens for styling
- ✅ Use `useState` for local state, Context for global state
- ✅ Use EventEmitter for cross-component notifications
- ✅ Handle errors with `.catch()` and `Emitter.emit('notification', ...)`
- ✅ Use `t()` function for all user-facing text
- ✅ Include ARIA labels and accessibility features
- ✅ Use TypeScript with proper types
- ✅ Test with `@testing-library/react` and `@testing-library/user-event`
- ✅ Follow existing patterns in StorageBrowser and Buckets components
- ✅ **Run `npm run format` after creating or modifying files** - Ensures consistent Prettier formatting across the codebase

**DON'T:**
- ❌ Use PatternFly 5 or hardcoded `pf-` classes (must be `pf-v6-`)
- ❌ Use hardcoded colors, sizes, or spacing (use `--pf-t--` tokens)
- ❌ Use legacy `--pf-v6-global--` tokens (use semantic `--pf-t--` tokens)
- ❌ Add React Query without discussing architecture
- ❌ Create service layer without planning
- ❌ Add authentication without discussing architecture
- ❌ Skip accessibility features
- ❌ Use inline styles unless absolutely necessary
- ❌ Use `alert()` or `console.error()` for user-facing errors (use EventEmitter)

### PatternFly 6 Guide

For comprehensive PatternFly 6 development guidance:
- **[Complete PF6 Guide](../docs/development/pf6-guide/README.md)** - Components, styling, testing patterns, and best practices
- **[Component Reference](../docs/development/pf6-guide/components/README.md)** - PatternFly 6 component usage
- **[Styling Standards](../docs/development/pf6-guide/guidelines/styling-standards.md)** - Design token usage and theming
- **[Testing Patterns](../docs/development/pf6-guide/testing-patterns/README.md)** - Testing guides for PatternFly 6 components
- **[Troubleshooting](../docs/development/pf6-guide/troubleshooting/README.md)** - Common issues and solutions

## 🔧 Known Limitations

- No authentication or role-based access
- No service layer (API calls embedded in components)
- Limited i18n (English only currently implemented)
- Ephemeral settings (not persisted unless from env vars)
- No global error boundary (component-level only)
- Minimal test coverage

## 🛠️ Debugging Workflow

1. **Make component changes** - Save the file
2. **Check Webpack dev server** - Webpack compiles automatically, check terminal output
3. **If TypeScript errors** - Fix types and save, Webpack will recompile
4. **If ESLint warnings** - Fix or add disable comment if intentional
5. **Check browser** - HMR should auto-update, check browser console for runtime errors
6. **If HMR fails** - Browser will show error overlay with details

**Primary debugging tool**: Browser DevTools Console (not log files)

## 📚 Related Documentation

- [Frontend Architecture](../docs/architecture/frontend-architecture.md) - Complete implementation details, component patterns, and examples
- [PatternFly 6 Guide](../docs/development/pf6-guide/README.md) - Comprehensive guide for frontend development
- [Development Workflow](../docs/development/development-workflow.md) - Build process, testing, and development setup
- [Data Flow](../docs/architecture/data-flow.md) - API communication and event patterns
- Root [CLAUDE.md](../CLAUDE.md) - Project overview
- Backend [CLAUDE.md](../backend/CLAUDE.md) - Backend API context

---
> Source: [rh-aiservices-bu/odh-tec](https://github.com/rh-aiservices-bu/odh-tec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
