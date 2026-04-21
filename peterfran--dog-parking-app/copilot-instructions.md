## dog-parking-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the frontend application for the Dog Parking platform - a Next.js 15 React application providing premium dog care services including daycare, boarding, grooming, and training. The app integrates with a separate AWS serverless backend API and uses Firebase for authentication.

## Visual Development

### Design Principles

- Comprehensive design checklist in `/context/design-principles.md`
- Brand style guide in `/context/style-guide.md`
- When making visual (front-end, UI/UX) changes, always refer to these files for guidance

### Browser Testing & Debugging

**Available Tools:**
- **Chrome DevTools MCP** - For performance analysis, debugging, and visual inspection
- **Playwright MCP** - For automated testing and reliable screenshots

**When to Use:**
Use browser tools when the user requests:
- Visual review or screenshots
- Performance analysis (Core Web Vitals, LCP, CLS, FCP)
- Debugging (console errors, network issues)
- Responsive design verification
- Accessibility checks

**Do not** automatically run browser tools after every change. Wait for explicit user request.

### Comprehensive Design Review

Invoke the `@agent-design-review` subagent for thorough design validation when the user requests:

- Comprehensive review of UI/UX features
- PR review with visual changes
- Accessibility and responsiveness testing

## Development Commands

**Development Server:**

```bash
npm run dev          # Start development server with Turbopack
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint (disabled during builds)
```

**Environment Setup:**

```bash
cp .env.example .env.local   # Copy environment template
# Edit .env.local with Firebase configuration and API URL
```

**Key Environment Variables:**

- `NEXT_PUBLIC_FIREBASE_API_KEY` - Firebase API key
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` - Firebase auth domain
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID` - Firebase project ID
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` - Firebase storage bucket
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` - Firebase messaging sender ID
- `NEXT_PUBLIC_FIREBASE_APP_ID` - Firebase app ID
- `NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID` - Firebase measurement ID (optional)
- `NEXT_PUBLIC_API_BASE_URL` - Backend API endpoint (AWS Lambda)
- `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST` - Development emulator (optional, use localhost:9099)

## Architecture Overview

### Authentication Architecture

- **Firebase Authentication** handles user registration, login, and JWT token management
- **AuthContext** (`src/contexts/AuthContext.tsx`) provides authentication state throughout the app
- **ProtectedRoute** component wraps authenticated pages and handles redirects
- JWT tokens are automatically included in API requests via the `apiClient`

### Data Flow Architecture

- **TanStack React Query** manages server state and API caching
- **useApi hooks** (`src/hooks/useApi.ts`) provide typed API methods with authentication
- **ApiClient** (`src/lib/api-client.ts`) handles HTTP requests with automatic token injection
- **Query keys** are centralized in `QueryKeys` constant for type-safe cache invalidation
- **Protected queries** automatically include Firebase JWT tokens and are disabled when user is not authenticated
- **Mutations** automatically invalidate related queries on success

### Styling Architecture

- **Tailwind CSS v4** is used for styling with CSS-based configuration
- **Custom color palette** defined in `src/app/globals.css` using the `@theme` directive
- **UI components** (`src/components/ui/`) use `class-variance-authority` for variant management with Tailwind classes
- Each UI component inlines the `cn` utility function to avoid import resolution issues
- Components use a warm, professional design with gentle gradients and thoughtful color choices
- **Design system colors**:
  - Primary: `#4f46e5` (warm indigo, friendlier than harsh navy)
  - Accent colors: `accent-sage` (`#dcfce7`), `accent-peach` (`#fed7aa`), `accent-lavender` (`#f3e8ff`)
  - Orange accent: `#f97316` (warmer than aggressive orange)
- **Important**: Tailwind v4 uses CSS-based config in `globals.css`, NOT `tailwind.config.ts`

### Tailwind CSS v4 Configuration

This project uses Tailwind CSS v4 (beta) which has a completely different configuration system:

**Color Configuration (in `src/app/globals.css`):**
```css
@theme {
  --color-primary: #4f46e5;
  --color-accent-sage: #dcfce7;
  --color-accent-peach: #fed7aa;
  --color-accent-lightBlue: #dbeafe;
  --color-accent-lavender: #f3e8ff;
  --color-accent-textGray: #6b7280;
  /* etc... */
}
```

**Key Differences from v3:**
- Configuration is CSS-based using `@theme` directive, not JavaScript config file
- Custom colors are defined as CSS variables in `globals.css`
- `tailwind.config.ts` is essentially empty (kept for compatibility only)
- PostCSS uses `@tailwindcss/postcss` plugin
- Build process compiles CSS variables into utility classes

**Adding New Colors:**
1. Add to `@theme` block in `globals.css` using `--color-` prefix
2. Use in components as `bg-yourColor`, `text-yourColor`, etc.
3. NO configuration needed in `tailwind.config.ts`

### Route Protection Pattern

```tsx
<ProtectedRoute>
  <MainLayout>
    <PageContent />
  </MainLayout>
</ProtectedRoute>
```

### API Integration Pattern

The app integrates with a separate AWS serverless backend:

- **Public endpoints**: `/venues`, `/venues/{id}`, `/venues/{id}/slots` (no authentication required)
- **Protected endpoints**: Owner profile, dog management (`/dogs/*`), bookings (`/bookings/*`) - all require Firebase JWT
- **Error handling**: API errors are logged and re-thrown with descriptive messages
- **Token management**: Bearer tokens are automatically injected via `Authorization` header
- **Request format**: All requests use `Content-Type: application/json`

### State Management Strategy

- **Authentication state**: Managed by AuthContext with Firebase Auth listener
- **Server state**: TanStack Query with optimistic updates and cache invalidation
- **Form state**: React Hook Form with Zod validation
- **Local UI state**: Standard React useState for component-specific state

## Key Patterns and Conventions

### Import Path Resolution

- **Relative imports** are used exclusively instead of `@/` path aliases
- This prevents Vercel build failures due to TypeScript path mapping issues
- Example: `import { useAuth } from '../../contexts/AuthContext';`

### Firebase Integration

- Firebase is initialized in `src/lib/firebase.ts` with emulator support for development
- AuthContext provides typed authentication methods: `signIn`, `signUp`, `signInWithGoogle`, `signOut`, `getIdToken`
- Auth emulator automatically connects when `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST` is set to `localhost:9099`
- Email verification is automatically sent on user registration
- Google Auth uses `select_account` prompt for better UX

### API Client Architecture

- Centralized `ApiClient` class handles all HTTP communication
- Automatic Bearer token injection for authenticated requests
- Type-safe methods for all backend endpoints
- Comprehensive error handling with typed error responses

### Component Architecture

- **Page components** use App Router pattern in `src/app/`
- **Layout components** provide consistent structure using `AppLayout` component
- **UI components** are self-contained with inlined utilities
- **Auth components** handle authentication flows and route protection

### Modal/Dialog Pattern

**IMPORTANT**: This project uses **Popover components** instead of Dialog components for modal interactions.

- **Correct Pattern**: Use `Popover`, `PopoverContent`, `PopoverTrigger` from `./ui/popover`
- **Incorrect Pattern**: Do NOT use `Dialog`, `DialogContent`, etc. from `./ui/dialog`
- **Example**: See `AuthModal.tsx` and `AddDogModal.tsx` for proper implementation
- **Structure**:
  ```tsx
  <Popover open={open} onOpenChange={setOpen}>
    <PopoverTrigger asChild>{children}</PopoverTrigger>
    <PopoverContent className="w-96 p-0" align="end">
      <Card className="border-0 shadow-lg">
        {/* Modal content */}
      </Card>
    </PopoverContent>
  </Popover>
  ```

### Build Configuration

- ESLint is disabled during builds (`ignoreDuringBuilds: true`) to prevent deployment failures
- Turbopack is used for faster development builds
- Next.js 15 with App Router and React 19
- **Tailwind CSS v4** with PostCSS plugin (`@tailwindcss/postcss`) for modern CSS processing

### Query Hook Patterns

When creating new API hooks, follow these patterns:

- **Public endpoints**: Use simple `useQuery` without authentication checks
- **Protected endpoints**: Include `getIdToken()` in `queryFn`, add `enabled: !!user` condition
- **Mutations**: Always invalidate related queries in `onSuccess` callback
- **Query keys**: Add new keys to centralized `QueryKeys` constant for type safety

### Authentication Flow Patterns

- All protected pages must be wrapped with `<ProtectedRoute>`
- Use `const { user, loading } = useAuth()` to access auth state
- Protected API calls automatically handle token injection via `getIdToken()`
- Auth loading states should be handled in components to prevent flickering

### Error Handling Conventions

- API errors are automatically logged in the `ApiClient.request()` method
- Component-level error handling should use React Query's error states
- Auth errors are caught and re-thrown in `AuthContext` methods with console logging
- All async operations should include proper error boundaries

## Development Context

The frontend application is designed to work with:

- **Backend API**: AWS Lambda functions with DynamoDB (separate repository)
- **Authentication**: Firebase Auth with social login support
- **Deployment**: Vercel with automatic GitHub integration
- **Development**: Local development with Firebase emulator support

This application prioritizes rapid deployment and development velocity while maintaining type safety and modern React patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterFran) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
