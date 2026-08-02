## s4

> Manages modal open/close state with consistent API.

# CLAUDE.md - S4 Frontend Context

> **Note for AI Assistants**: This file contains AI-specific development context for the S4 React frontend. For user-facing architecture documentation, see [docs/architecture/frontend.md](../docs/architecture/frontend.md) and [docs/development/frontend.md](../docs/development/frontend.md). For project overview, see root [CLAUDE.md](../CLAUDE.md). For backend context, see [backend/CLAUDE.md](../backend/CLAUDE.md).

## Frontend Overview

**s4-frontend** - React 18 application with TypeScript, Webpack, and PatternFly 6 component library.

**Technology Stack**: React 18, PatternFly 6, React Router v7, TypeScript, Webpack
**Development**: Port 9000 with Webpack HMR (Hot Module Replacement)
**Production**: Built and served statically by Fastify backend on port 5000 (container)

**For detailed architecture**, see [docs/architecture/frontend.md](../docs/architecture/frontend.md).

## 🎨 PatternFly 6 Critical Requirements

⚠️ **MANDATORY**: Follow the [PatternFly 6 Development Guide](../docs/development/pf6-guide/README.md) as the **AUTHORITATIVE SOURCE** for all UI development.

### Context7 Warning

**DO NOT use Context7 for PatternFly components.** Context7 may contain outdated PatternFly versions (v5 or earlier) that conflict with this project's PatternFly 6 requirements.

**Instead use:**

- **Local guide**: [`docs/development/pf6-guide/`](../docs/development/pf6-guide/README.md) (authoritative, project-specific)
- **Official docs**: [PatternFly.org](https://www.patternfly.org/) (always up-to-date)

Context7 is fine for non-PatternFly libraries: React, Axios, React Router, Jest, i18next, and other dependencies.

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
- **Context for Global**: AuthContext for authentication state
- **EventEmitter for Cross-Component**: Use emitter for decoupled communication (upload progress, notifications)
- **Reusable Hooks**: Use custom hooks (`useModal`, `useStorageLocations`) for common patterns
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

### Component Utilities

6. **Hooks**: MUST use reusable hooks for common patterns

   - Use `useModal()` for modal state management instead of inline useState
   - Use `useStorageLocations()` for loading storage locations

7. **Notifications**: MUST use notification utilities for user feedback

   - Import from `@app/utils/notifications`
   - Use `notifySuccess()`, `notifyError()`, `notifyWarning()`, `notifyInfo()`
   - Use `notifyApiError()` for consistent API error handling

8. **Validation**: MUST use validation utilities for input validation
   - Import from `@app/utils/validation`
   - Use `validateS3BucketName()` for bucket name validation
   - Use `validateS3ObjectName()` for object/folder name validation
   - Use helper functions like `getBucketNameRules()` to show validation rules to users

### Component Pattern Example

```tsx
import React from 'react';
import { useModal } from '@app/hooks';
import { notifySuccess, notifyError, notifyApiError } from '@app/utils/notifications';
import { validateS3BucketName } from '@app/utils/validation';
import apiClient from '@app/utils/apiClient';

const MyComponent: React.FC = () => {
  const [loading, setLoading] = React.useState(false);
  const createModal = useModal();
  const [bucketName, setBucketName] = React.useState('');

  const handleCreate = async () => {
    // Validate input
    if (!validateS3BucketName(bucketName)) {
      notifyError('Invalid bucket name', 'Please check bucket naming rules');
      return;
    }

    setLoading(true);
    try {
      const response = await apiClient.post('/api/buckets', { bucketName });
      notifySuccess('Bucket created', `Bucket ${bucketName} created successfully`);
      createModal.close();
    } catch (error) {
      notifyApiError('Create bucket', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <Button onClick={createModal.open}>Create Bucket</Button>
      <Modal isOpen={createModal.isOpen} onClose={createModal.close}>
        {/* Modal content */}
      </Modal>
    </div>
  );
};
```

## 🎣 Reusable Hooks

S4 provides custom hooks for common UI patterns to reduce code duplication and ensure consistency.

### useModal Hook

Manages modal open/close state with consistent API.

**Location**: `src/app/hooks/useModal.ts`

**Usage**:

```tsx
import { useModal } from '@app/hooks';

const MyComponent: React.FC = () => {
  const createModal = useModal();
  const deleteModal = useModal();

  return (
    <>
      <Button onClick={createModal.open}>Create</Button>
      <Modal isOpen={createModal.isOpen} onClose={createModal.close}>
        {/* Modal content */}
      </Modal>

      <Button onClick={deleteModal.open}>Delete</Button>
      <Modal isOpen={deleteModal.isOpen} onClose={deleteModal.close}>
        {/* Modal content */}
      </Modal>
    </>
  );
};
```

**API**:

- `isOpen: boolean` - Current modal state
- `open: () => void` - Open the modal
- `close: () => void` - Close the modal
- `toggle: () => void` - Toggle modal state

### useStorageLocations Hook

Loads and manages storage location state with automatic error handling and notifications.

**Location**: `src/app/hooks/useStorageLocations.ts`

**Usage**:

```tsx
import { useStorageLocations } from '@app/hooks';

const MyComponent: React.FC = () => {
  const { locations, loading, error, refreshLocations } = useStorageLocations();

  if (loading) return <Spinner />;
  if (error) return <Alert variant="danger" title={error} />;

  return (
    <>
      {locations.map((location) => (
        <div key={location.id}>
          {location.name} - {location.available ? 'Available' : 'Unavailable'}
        </div>
      ))}
      <Button onClick={refreshLocations}>Refresh</Button>
    </>
  );
};
```

**API**:

- `locations: StorageLocation[]` - Array of storage locations
- `loading: boolean` - Loading state
- `error: string | null` - Error message if loading failed
- `refreshLocations: () => Promise<void>` - Refresh locations from backend

**Features**:

- Automatic loading on mount
- Integrated notification system (warns if no locations or all unavailable)
- Error handling with user-friendly messages
- Refresh capability

## 📢 Notification Utilities

S4 provides centralized notification utilities for consistent user feedback across the application.

**Location**: `src/app/utils/notifications.ts`

### Basic Notifications

```tsx
import { notifySuccess, notifyError, notifyWarning, notifyInfo } from '@app/utils/notifications';

// Success notification
notifySuccess('Operation successful', 'Your file has been uploaded');

// Error notification
notifyError('Operation failed', 'Unable to delete bucket');

// Warning notification
notifyWarning('Low disk space', 'Consider cleaning up old files');

// Info notification
notifyInfo('Maintenance scheduled', 'System will be down at midnight');
```

### API Error Handling

The `notifyApiError()` function provides consistent error handling for API calls:

```tsx
import { notifyApiError } from '@app/utils/notifications';
import apiClient from '@app/utils/apiClient';

try {
  await apiClient.post('/api/buckets', { bucketName });
} catch (error) {
  // Automatically extracts error message from Axios error response
  // Shows user-friendly notification with proper error details
  notifyApiError('Create bucket', error);
}
```

**Features**:

- Handles Axios errors with response data extraction
- Handles generic Error objects
- Handles unknown error types
- Logs errors to console for debugging
- Shows user-friendly notification via EventEmitter

### Integration with EventEmitter

All notification utilities use the EventEmitter system internally:

```tsx
// notifications.ts internally calls:
Emitter.emit('notification', {
  variant: 'success',
  title: 'Title',
  description: 'Description',
});

// AppLayout.tsx listens for these events and displays toast notifications
```

## ✅ Validation Utilities

S4 provides comprehensive validation utilities for S3 bucket and object names.

**Location**: `src/app/utils/validation.ts`

### Bucket Name Validation

```tsx
import { validateS3BucketName, getBucketNameRules } from '@app/utils/validation';

// Validate bucket name
const isValid = validateS3BucketName('my-bucket');

// Validate with duplicate checking
const existingBuckets = ['bucket1', 'bucket2'];
const isValid = validateS3BucketName('new-bucket', existingBuckets);

// Get validation rules to display to user
const rules = getBucketNameRules();
// Returns array of rule strings like:
// - "Bucket names must be between 3 and 63 characters long"
// - "Bucket names can consist only of lowercase letters, numbers, dots (.), and hyphens (-)"
```

**Validation Rules** (AWS-compliant):

- 3-63 characters long
- Only lowercase letters, numbers, dots (.), and hyphens (-)
- Must start and end with letter or number
- No consecutive periods
- Not formatted as IP address
- No duplicates in existing buckets

### Object/Folder Name Validation

```tsx
import { validateS3ObjectName, getObjectNameRules, getFolderNameRules } from '@app/utils/validation';

// Validate S3 object name
const isValid = validateS3ObjectName('models/llama/config.json');

// Validate folder name (with storage-type-specific rules)
const isValidLocal = validateS3ObjectName('my-folder', 'local'); // More permissive for local storage
const isValidS3 = validateS3ObjectName('my-folder', 's3'); // Stricter for S3

// Get validation rules to display
const objectRules = getObjectNameRules();
const folderRules = getFolderNameRules('s3'); // or 'local'
```

**Common Validation Rules**:

- Cannot be empty
- Cannot contain null characters
- Cannot be `.` or `..`
- Cannot start with `../`

### Usage in Forms

```tsx
import { validateS3BucketName, getBucketNameRules } from '@app/utils/validation';
import { notifyError } from '@app/utils/notifications';

const CreateBucketModal: React.FC = () => {
  const [bucketName, setBucketName] = useState('');
  const [showRules, setShowRules] = useState(false);

  const handleSubmit = () => {
    if (!validateS3BucketName(bucketName)) {
      notifyError('Invalid bucket name', 'Please check the bucket naming rules');
      return;
    }
    // Proceed with creation
  };

  return (
    <Form>
      <FormGroup label="Bucket name" isRequired>
        <TextInput
          value={bucketName}
          onChange={(_, value) => setBucketName(value)}
          onFocus={() => setShowRules(true)}
          validated={bucketName && !validateS3BucketName(bucketName) ? 'error' : 'default'}
        />
        {showRules && (
          <FormHelperText>
            <HelperText>
              <HelperTextItem>
                Bucket naming rules:
                <ul>
                  {getBucketNameRules().map((rule) => (
                    <li key={rule}>{rule}</li>
                  ))}
                </ul>
              </HelperTextItem>
            </HelperText>
          </FormHelperText>
        )}
      </FormGroup>
    </Form>
  );
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

**For complete workflow**, see [docs/development/frontend.md](../docs/development/frontend.md).

## 📁 Component Organization

Main components in `src/app/components/`:

- **AppLayout** - Main layout with navigation sidebar
- **AuthContext** - Authentication state management (JWT tokens, login/logout)
- **AuthGate** - Route protection wrapper that enforces authentication
- **Login** - Login form component
- **ProtectedRoute** - HOC for protecting routes (requires authentication)
- **StorageBrowser** - Unified storage browser for S3 and local storage with upload/download
- **Buckets** - S3 bucket management (includes BucketNotificationsModal for webhook configuration)
- **Settings** - S3 connection configuration
- **DocumentRenderer** - Markdown/document viewer
- **ResponsiveTable** - Responsive table wrapper with mobile card view
- **StorageRouteGuard** - Route guard checking storage availability
- **Transfer** - Transfer management components (TransferAction, TransferProgress)
- **NotFound** - 404 page

Key utilities in `src/app/hooks/`:

- **useModal** - Reusable modal state management hook
- **useStorageLocations** - Storage location loading and status checking hook
- **useIsMobile** - Responsive breakpoint detection hook

Key utilities in `src/app/utils/`:

- **apiClient** - Centralized axios instance with JWT auth and 401 handling
- **sseTickets** - One-time ticket utilities for secure SSE authentication
- **EventEmitter** - Cross-component communication (notifications, upload progress)
- **notifications** - Helper functions for toast notifications (`notifySuccess`, `notifyError`, `notifyWarning`, `notifyInfo`, `notifyApiError`)
- **validation** - S3 bucket and object name validation utilities

Key utilities in `src/app/services/`:

- **storageService** - Centralized storage API service (unified S3 + local interface)

**For component patterns and examples**, see [docs/architecture/frontend.md](../docs/architecture/frontend.md).

## 🌐 Routing

- React Router v7 for navigation
- Routes defined in `src/app/routes.tsx`
- Main routes: `/browse/:locationId?/:path?` (Storage Browser), `/buckets`, `/settings`
- **Protected routes**: All routes except `/login` require authentication when enabled
- **AuthGate wrapper**: Checks auth status on mount and protects all routes

### Authentication Flow

**On Application Load**:

1. `AuthGate` component wraps all routes
2. Calls `GET /api/auth/info` to check if auth is required
3. If auth disabled, renders app immediately
4. If auth enabled, checks for JWT token in sessionStorage
5. If token exists, validates via `GET /api/auth/me`; if invalid, redirects to `/login`

**Token Management**:

- JWT stored in sessionStorage (cleared on tab close)
- `apiClient` (axios instance) includes token in `Authorization: Bearer <token>` header
- `apiClient` intercepts 401 responses → emits `auth:unauthorized` → `AuthContext` triggers logout

**SSE Authentication** (`src/app/utils/sseTickets.ts`):

Use `createAuthenticatedEventSource()` for all SSE connections — never put JWT tokens in URLs:

```tsx
import { createAuthenticatedEventSource } from '@app/utils/sseTickets';

// For transfer progress
const eventSource = await createAuthenticatedEventSource(jobId, 'transfer');
eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // Handle progress updates
};

// For upload progress
const eventSource = await createAuthenticatedEventSource(encodedKey, 'upload');
```

See [backend/CLAUDE.md](../backend/CLAUDE.md) for authentication implementation details (JWT flow, SSE ticket system, rate limiting).

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

**For full details**, see [docs/architecture/frontend.md](../docs/architecture/frontend.md).

## 🔌 API Integration

**Backend API**: Fastify server (port 8888 in dev, port 5000 in container)

**API Client**: Centralized axios instance (`src/app/utils/apiClient.ts`)

- Automatically includes JWT token in `Authorization: Bearer <token>` header
- Intercepts 401 responses and emits `auth:unauthorized` event
- CORS configured with credentials support

**API Pattern**: Direct axios calls in components using `apiClient`

```tsx
import apiClient from '@app/utils/apiClient';

const response = await apiClient.get('/api/buckets');
```

See [backend/CLAUDE.md](../backend/CLAUDE.md) for complete endpoint listing, or [docs/api/](../docs/api/) for API reference.

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

### CSS Architecture

S4 uses a layered CSS architecture:

1. **PatternFly 6 Design Tokens** (Foundation)

   - Semantic tokens with `--pf-t--` prefix
   - Auto-adapt to light/dark themes
   - Never hardcode values

2. **CSS Variables** (Application-level)

   - Custom variables in `app.css` for app-specific values
   - Modal widths, form dimensions, viewer heights
   - Use `--s4-*` prefix for custom variables

3. **Utility Classes** (Component-level)
   - Reusable spacing, layout, text utilities
   - Consistent with PatternFly patterns
   - Reduce inline styles

### CSS Variables

**Location**: `frontend/src/app/app.css`

```css
:root {
  /* Modal widths (responsive) */
  --s4-modal-width-small: min(400px, 95vw);
  --s4-modal-width-standard: min(600px, 90vw);
  --s4-modal-width-large: min(900px, 85vw);

  /* Form dimensions (responsive) */
  --s4-form-width-quarter: min(25%, 100%);
  --s4-form-width-half: min(50%, 100%);
  --s4-form-width-three-quarter: min(75%, 100%);
  --s4-form-input-width: min(250px, 100%);
  --s4-form-select-width-sm: min(100px, 100%);
  --s4-form-select-width-md: min(400px, 100%);
  --s4-form-select-width-lg: min(500px, 100%);

  /* Accessibility */
  --s4-touch-target-min: 44px;

  /* Icon sizes */
  --s4-icon-size-sm: 16px;
  --s4-icon-size-md: 20px;

  /* Viewer heights */
  --s4-viewer-height: 70vh;
}
```

### Utility Classes

**Location**: `frontend/src/app/app.css` (lines 217-372)

**Spacing Utilities**:

```css
.pf-u-margin-bottom-md {
  margin-bottom: var(--pf-t--global--spacer--md);
}
.pf-u-padding-md {
  padding: var(--pf-t--global--spacer--md);
}
```

**Layout Utilities**:

```css
.pf-u-flex-column {
  display: flex;
  flex-direction: column;
}
.pf-u-full-height {
  height: 100%;
}
```

**Text Utilities**:

```css
.pf-u-text-subtle {
  color: var(--pf-t--global--text--color--subtle);
}
.pf-u-text-center {
  text-align: center;
}
```

**Icon Utilities**:

```css
.pf-u-icon-sm {
  height: var(--s4-icon-size-sm);
}
.pf-u-icon-md {
  height: var(--s4-icon-size-md);
}
```

### Styling Best Practices

- Use PatternFly 6 design tokens exclusively
- Support dark theme with semantic tokens (they auto-adapt)
- Avoid hardcoded values - use `--pf-t--` tokens
- Use CSS variables for app-specific repeated values
- Use utility classes to reduce inline styles
- Test in both light and dark themes

**Example**:

```css
/* ✅ CORRECT - Semantic token */
.my-element {
  color: var(--pf-t--global--color--brand--default);
  padding: var(--pf-t--global--spacer--md);
  width: var(--s4-modal-width-standard);
}

/* ❌ WRONG - Hardcoded value */
.my-element {
  color: #0066cc;
  padding: 16px;
  width: 500px;
}
```

**Inline Styles**: Avoid unless absolutely necessary (dynamic values that can't be classes)

## 📚 Project Structure

```
frontend/
├── src/
│   ├── app/
│   │   ├── components/      # React components
│   │   ├── hooks/           # Custom hooks (useModal, useStorageLocations, useIsMobile)
│   │   ├── services/        # API service layer (storageService)
│   │   ├── utils/           # Utilities (apiClient, EventEmitter, notifications, validation)
│   │   ├── assets/          # Images and icons
│   │   ├── routes.tsx       # Route definitions
│   │   ├── config.tsx       # App configuration
│   │   └── index.tsx        # App component
│   ├── i18n/                # Internationalization
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
- ✅ Use `useModal()` hook for modal state management
- ✅ Use `useStorageLocations()` hook for loading storage locations
- ✅ Use notification utilities (`notifySuccess`, `notifyError`, `notifyApiError`) instead of manual Emitter.emit
- ✅ Use validation utilities (`validateS3BucketName`, `validateS3ObjectName`) for input validation
- ✅ Use CSS utility classes from app.css to reduce inline styles
- ✅ Use `--s4-*` CSS variables for app-specific repeated values
- ✅ Handle errors with `.catch()` and notification utilities
- ✅ Use `t()` function for all user-facing text
- ✅ Include ARIA labels and accessibility features
- ✅ Use TypeScript with proper types
- ✅ Test with `@testing-library/react` and `@testing-library/user-event`
- ✅ Follow existing patterns in StorageBrowser and Buckets components
- ✅ Use `createAuthenticatedEventSource()` for all SSE connections (transfer and upload progress)
- ✅ **Run `npm run format` after creating or modifying files** - Ensures consistent Prettier formatting across the codebase

**DON'T:**

- ❌ Use PatternFly 5 or hardcoded `pf-` classes (must be `pf-v6-`)
- ❌ Use hardcoded colors, sizes, or spacing (use `--pf-t--` tokens)
- ❌ Use legacy `--pf-v6-global--` tokens (use semantic `--pf-t--` tokens)
- ❌ Create manual modal state with useState (use `useModal()` hook)
- ❌ Manually call `Emitter.emit('notification', ...)` (use notification utilities)
- ❌ Write duplicate validation logic (use validation utilities)
- ❌ Use inline styles for spacing/layout (use utility classes)
- ❌ Hardcode repeated dimensions (use CSS variables)
- ❌ Add state management libraries without discussing architecture
- ❌ Modify authentication flow without understanding AuthContext and apiClient
- ❌ Put JWT tokens in SSE URLs (use one-time tickets via `createAuthenticatedEventSource()`)
- ❌ Create EventSource manually (use `createAuthenticatedEventSource()` instead)
- ❌ Skip accessibility features
- ❌ Use `alert()` or `console.error()` for user-facing errors (use notification utilities)
- ❌ Use Context7 for PatternFly documentation (use local `docs/development/pf6-guide/` and PatternFly.org instead)

### PatternFly 6 Guide

For comprehensive PatternFly 6 development guidance:

- **[Complete PF6 Guide](../docs/development/pf6-guide/README.md)** - Components, styling, testing patterns, and best practices
- **[Component Reference](../docs/development/pf6-guide/components/README.md)** - PatternFly 6 component usage
- **[Styling Standards](../docs/development/pf6-guide/guidelines/styling-standards.md)** - Design token usage and theming
- **[Testing Patterns](../docs/development/pf6-guide/testing-patterns/README.md)** - Testing guides for PatternFly 6 components
- **[Troubleshooting](../docs/development/pf6-guide/troubleshooting/README.md)** - Common issues and solutions

## 🔧 Known Limitations

- Simple authentication only (single admin user, no multi-user or role-based access)
- i18n translations may be incomplete or imperfect for non-English languages
- Ephemeral settings (not persisted unless from env vars)
- No global error boundary (component-level only)
- Minimal test coverage
- JWT tokens stored in sessionStorage (cleared on tab close)

## 🛠️ Debugging Workflow

1. **Make component changes** - Save the file
2. **Check Webpack dev server** - Webpack compiles automatically, check terminal output
3. **If TypeScript errors** - Fix types and save, Webpack will recompile
4. **If ESLint warnings** - Fix or add disable comment if intentional
5. **Check browser** - HMR should auto-update, check browser console for runtime errors
6. **If HMR fails** - Browser will show error overlay with details

**Primary debugging tool**: Browser DevTools Console (not log files)

## 📚 Related Documentation

### For AI Assistants

- Root [CLAUDE.md](../CLAUDE.md) - Project overview and AI development context
- Backend [CLAUDE.md](../backend/CLAUDE.md) - Backend API AI context

### For Users and Developers

- [Frontend Architecture](../docs/architecture/frontend.md) - Complete implementation details, component patterns, and examples
- [Frontend Development](../docs/development/frontend.md) - Build process, testing, and development setup
- [PatternFly 6 Guide](../docs/development/pf6-guide/README.md) - Comprehensive guide for frontend development

---
> Source: [rh-aiservices-bu/s4](https://github.com/rh-aiservices-bu/s4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
