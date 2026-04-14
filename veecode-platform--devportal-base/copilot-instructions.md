## devportal-base

> Backstage-based developer portal using monorepo structure with Yarn workspaces

# Windsurf Rules for DevPortal Base

## Project Type

Backstage-based developer portal using monorepo structure with Yarn workspaces

## Code Style & Standards

### TypeScript

- Use strict TypeScript - avoid `any` without justification
- Prefer interfaces over types for object shapes
- Use proper type imports: `import type { ... }`

### React Patterns

- Use functional components with hooks
- Implement proper error boundaries
- Use React.memo for expensive renders
- Lazy load heavy components with `React.lazy()`

### Styling

- Use MUI v5 components from `@mui/material`
- For legacy code, use `makeStyles` from `@mui/styles` (v4 compatibility)
- For new code, prefer `sx` prop or `styled()` from `@mui/material/styles`
- Never mix styling approaches in the same component

### Import Order

1. React and external libraries
2. Backstage core imports
3. MUI imports
4. Internal/local imports
5. Types (with `import type`)
6. Styles

Example:

```typescript
import React from 'react';
import { useApi } from '@backstage/core-plugin-api';
import { Button, Box } from '@mui/material';
import { MyLocalComponent } from './MyLocalComponent';
import type { MyType } from './types';
```

## File Organization

### Plugin Structure

```
plugins/my-plugin/
├── src/
│   ├── plugin.ts           # Plugin definition
│   ├── routes.ts           # Route references
│   ├── components/         # React components
│   │   └── MyPage/
│   │       ├── MyPage.tsx
│   │       └── index.ts
│   └── index.ts           # Public exports
├── dev/                   # Standalone dev environment
└── package.json
```

### Component Files

- One component per file
- Co-locate tests: `MyComponent.test.tsx`
- Export from index: `index.ts`
- Keep files under 300 lines

## Testing Requirements

### Unit Tests

- Write tests for all new components
- Use `@backstage/test-utils` for rendering
- Test user interactions, not implementation details
- Aim for meaningful coverage, not 100%

### Test Pattern

```typescript
import { renderInTestApp } from '@backstage/test-utils';

describe('MyComponent', () => {
  it('should render correctly', async () => {
    const { getByText } = await renderInTestApp(<MyComponent />);
    expect(getByText('Expected Text')).toBeInTheDocument();
  });
});
```

## Plugin Development

### Creating New Plugins

1. Use Backstage CLI: `yarn backstage-cli create-plugin`
2. Follow naming convention: `@internal/plugin-{name}`
3. Add to workspace in root `package.json`
4. Export properly from `src/index.ts`

### Dynamic Plugins

Backstage plugins are usually developed in a separate repository, packaged and published into a NPM registry.

Newer plugins are distributed with dynamic packaging. They are defined in the `/dynamic-plugins/downloads/plugins.json` file and the build process downloads the plugins from the NPM registry, unpacks them and places them in the `/dynamic-plugins-root/` directory.

Older plugins not distributed with dynamic packaging, so we need to create a wrapper for them. The wrapper is a simple script that exports the plugin as a dynamic plugin. All wrapped plugins are located in the `/dynamic-plugins/wrappers/` directory.

We explain more about plugins and the build process in the `docs/` directory, see `docs/DYNAMIC_PLUGINS_ARCHITECTURE.md` and `docs/PLUGINS.md`.

## Configuration

### Secrets Management

- **NEVER** commit secrets to git
- Use environment variables for sensitive data
- Use `app-config.local.yaml` for local secrets (gitignored)
- Document required environment variables

### Config Files

- `app-config.yaml` - Base configuration (committed)
- `app-config.production.yaml` - Production overrides (ignore for now, not maintained)
- `app-config.local.yaml` - Local development (gitignored)

## Common Commands

```bash
# Building
yarn install
yarn build                  # Build all packages

# Development
yarn dev-local                    # Start frontend and backend
LOG_LEVEL=debug yarn dev-local    # Start frontend and backend with debug logging

# Testing
yarn test                  # Run all tests
yarn test --watch          # Watch mode
yarn e2e                   # E2E tests

# Code Quality
yarn lint                  # Lint code
yarn lint:fix             # Fix lint issues
```

## Known Issues & Workarounds

### MUI v5 Migration

- Currently using `@mui/styles` for v4 compatibility
- MUI v4 compatibility is important for old plugins
- TablePagination values must be: 5, 10, 20, 50, 100
- Dark mode font colors fixed in Backstage 1.44.1+

### Dynamic Plugins

- Plugins must have valid `plugin-manifest.json`
- Scalprum name must match configuration exactly
- Check `dynamic-plugins-root/` for runtime plugins

## Documentation

### When to Update Docs

- New features or plugins
- Configuration changes
- Breaking changes
- Architecture decisions

### Doc Locations

- `/docs/` - Project documentation
- Plugin README - Plugin-specific docs
- Inline comments - Complex logic only

## Performance Guidelines

### Bundle Size

- Monitor bundle size with `yarn workspace app analyze`
- Lazy load routes and heavy components
- Use dynamic imports for large dependencies

### Optimization

- Use `React.memo` for expensive renders
- Implement proper loading states
- Avoid unnecessary re-renders
- Use `useMemo` and `useCallback` appropriately

## Security

### Best Practices

1. Validate all user inputs
2. Use RBAC for authorization
3. Sanitize displayed data
4. Keep dependencies updated
5. Review security advisories

### RBAC

- Configure in `rbac-policy.csv`
- Use permission checks in components
- Document required permissions

## Git Workflow

### Commits

- Write descriptive commit messages
- Keep commits atomic and focused
- Reference issues when applicable

### Branches

- Create feature branches from main
- Use descriptive branch names
- Rebase before merging

## Error Handling

### Component Errors

- Implement error boundaries
- Show user-friendly error messages
- Log errors for debugging
- Provide recovery options when possible

### API Errors

- Handle network failures gracefully
- Show loading states
- Provide retry mechanisms
- Log errors with context

## Accessibility

### Requirements

- Use semantic HTML
- Provide ARIA labels where needed
- Ensure keyboard navigation works
- Test with screen readers
- Maintain color contrast ratios

## When in Doubt

1. Check existing patterns in the codebase
2. Review `/docs/` directory
3. Follow Backstage documentation
4. Ask for clarification rather than guessing
5. Prefer simple solutions over complex ones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/veecode-platform) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
