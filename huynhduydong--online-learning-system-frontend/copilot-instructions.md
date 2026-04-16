## online-learning-system-frontend

> app/                    # Next.js App Router pages

# Project Rules - Online Learning System Frontend

## 📁 Project Structure Rules

### Directory Organization
```
app/                    # Next.js App Router pages
├── [locale]/          # Internationalized routes
├── api/               # API routes
└── globals.css        # Global styles

components/            # Reusable UI components
├── ui/               # shadcn/ui components
└── *.tsx             # Custom components

lib/                  # Utility libraries
├── api/              # API client and services
├── validations/      # Form validation schemas
├── config.ts         # App configuration
└── utils.ts          # Utility functions

hooks/                # Custom React hooks
contexts/             # React context providers
messages/             # i18n translation files
public/               # Static assets
src/                  # Additional source code
├── components/       # Legacy components (migrate to /components)
├── hooks/           # Legacy hooks (migrate to /hooks)
├── lib/             # Legacy lib (migrate to /lib)
├── mocks/           # MSW mock handlers
└── stores/          # State management stores

tests/               # Test files
├── e2e/            # End-to-end tests
└── unit/           # Unit tests
```

## 🎯 Coding Standards

### TypeScript Rules
- **ALWAYS** use TypeScript for all files
- **ALWAYS** define proper interfaces for API responses
- **NEVER** use `any` type - use proper typing or `unknown`
- **ALWAYS** export types and interfaces from appropriate files
- Use strict TypeScript configuration

### Component Rules
- **ALWAYS** use functional components with hooks
- **ALWAYS** use PascalCase for component names
- **ALWAYS** export components as default exports
- **ALWAYS** define prop interfaces for components
- Use React.FC type annotation for components

### File Naming Conventions
- Components: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- Hooks: `use-kebab-case.ts` (e.g., `use-dashboard.ts`)
- Utilities: `kebab-case.ts` (e.g., `api-client.ts`)
- Pages: `kebab-case` folders with `page.tsx`
- Types: `types.ts` or `*.types.ts`

## 🔌 API Integration Rules

### API Client Structure
```typescript
// lib/api/client.ts - Base API client
// lib/api/types.ts - All API type definitions
// lib/api/[service].ts - Service-specific API calls
```

### API Response Handling
- **ALWAYS** define interfaces for API responses
- **ALWAYS** handle both old and new API structures for backward compatibility
- **ALWAYS** use proper error handling with try-catch blocks
- **ALWAYS** log errors for debugging

### API Response Mapping Pattern
```typescript
// For new API responses with data wrapper
interface NewApiResponse {
  data: {
    // actual data
  }
  message: string
  success: boolean
}

// For legacy API responses
interface LegacyApiResponse {
  // direct data structure
  success: boolean
}

// Always provide backward compatibility
async function apiMethod(): Promise<InternalType> {
  try {
    const response = await client.get<NewApiResponse>('/endpoint')
    
    if (response.success && response.data) {
      // Map new API structure to internal format
      return mapToInternalFormat(response.data)
    }
    
    throw new Error('API call failed')
  } catch (error) {
    console.error('API error:', error)
    throw error
  }
}
```

## 🎨 UI/UX Rules

### Component Library
- **ALWAYS** use shadcn/ui components from `components/ui/`
- **ALWAYS** follow shadcn/ui patterns for new components
- **NEVER** create custom UI components if shadcn/ui equivalent exists
- Use Tailwind CSS for styling

### Styling Rules
- **ALWAYS** use Tailwind CSS classes
- **NEVER** use inline styles or CSS modules
- Use CSS variables for theme colors
- Follow mobile-first responsive design

### Accessibility Rules
- **ALWAYS** include proper ARIA labels
- **ALWAYS** ensure keyboard navigation works
- **ALWAYS** use semantic HTML elements
- **ALWAYS** test with screen readers

## 🌐 Internationalization Rules

### i18n Structure
```
messages/
├── en.json           # English translations
└── vi.json           # Vietnamese translations
```

### Translation Rules
- **ALWAYS** use translation keys for all user-facing text
- **NEVER** hardcode text strings in components
- Use nested keys for organization: `"auth.login.title"`
- Keep translation files synchronized

### Usage Pattern
```typescript
import { useTranslations } from 'next-intl'

function Component() {
  const t = useTranslations('namespace')
  
  return <h1>{t('title')}</h1>
}
```

## 🔐 Authentication Rules

### Auth Context Pattern
- **ALWAYS** use AuthContext for authentication state
- **ALWAYS** protect routes with authentication checks
- **ALWAYS** handle token refresh automatically
- Store tokens securely (not in localStorage for sensitive data)

### Protected Routes
```typescript
// Use middleware.ts for route protection
// Check authentication in layout.tsx for protected sections
```

## 📊 State Management Rules

### State Management Strategy
- Use React Context for global state (auth, theme)
- Use custom hooks for component-specific state
- Use Zustand stores for complex state management
- **NEVER** prop drill more than 2 levels

### Custom Hooks Pattern
```typescript
// hooks/use-feature.ts
export function useFeature() {
  const [state, setState] = useState()
  
  const actions = useMemo(() => ({
    // action methods
  }), [])
  
  return { state, ...actions }
}
```

## 🧪 Testing Rules

### Test Structure
```
tests/
├── e2e/              # Playwright tests
│   └── *.spec.ts
└── unit/             # Jest tests
    ├── components/
    └── stores/
```

### Testing Requirements
- **ALWAYS** write unit tests for utilities and hooks
- **ALWAYS** write integration tests for API services
- **ALWAYS** write E2E tests for critical user flows
- Use MSW for API mocking in tests

## 📦 Dependencies Rules

### Package Management
- **ALWAYS** use pnpm for package management
- **ALWAYS** pin exact versions for critical dependencies
- **NEVER** install packages without team approval
- Keep dependencies up to date regularly

### Import Rules
- **ALWAYS** use absolute imports for internal modules
- **ALWAYS** group imports: external, internal, relative
- **ALWAYS** sort imports alphabetically within groups

```typescript
// External imports
import React from 'react'
import { NextPage } from 'next'

// Internal imports
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/use-auth'

// Relative imports
import './styles.css'
```

## 🚀 Performance Rules

### Code Splitting
- **ALWAYS** use dynamic imports for large components
- **ALWAYS** lazy load non-critical components
- Use Next.js built-in optimizations

### Image Optimization
- **ALWAYS** use Next.js Image component
- **ALWAYS** provide alt text for images
- Use appropriate image formats (WebP, AVIF)

## 🔧 Development Rules

### Environment Setup
- **ALWAYS** use `.env.local` for local environment variables
- **NEVER** commit sensitive environment variables
- Use `.env.local.example` for documentation

### Git Rules
- **ALWAYS** use conventional commit messages
- **ALWAYS** create feature branches from main
- **ALWAYS** run tests before committing
- Use Husky hooks for pre-commit checks

### Code Quality
- **ALWAYS** run ESLint and Prettier before committing
- **ALWAYS** fix all TypeScript errors
- **ALWAYS** maintain 80%+ test coverage
- Use meaningful variable and function names

## 📝 Documentation Rules

### Code Documentation
- **ALWAYS** add JSDoc comments for public APIs
- **ALWAYS** document complex business logic
- **ALWAYS** update README.md for setup changes
- Keep API documentation up to date

### Component Documentation
```typescript
/**
 * User profile component that displays user information
 * @param user - User object containing profile data
 * @param onEdit - Callback function when edit button is clicked
 */
interface UserProfileProps {
  user: User
  onEdit: () => void
}
```

## 🔄 Migration Rules

### Legacy Code Migration
- **GRADUALLY** migrate from `/src` to root-level directories
- **ALWAYS** maintain backward compatibility during migration
- **ALWAYS** update imports when moving files
- Document migration progress

### API Migration
- **ALWAYS** support both old and new API formats during transition
- **ALWAYS** create mapping functions for data transformation
- **ALWAYS** test both API formats thoroughly
- Remove legacy support only after full migration

## ⚠️ Error Handling Rules

### Error Boundaries
- **ALWAYS** wrap pages in error boundaries
- **ALWAYS** provide user-friendly error messages
- **ALWAYS** log errors for debugging
- Provide fallback UI for errors

### API Error Handling
```typescript
try {
  const result = await apiCall()
  return result
} catch (error) {
  console.error('API Error:', error)
  
  // Handle specific error types
  if (error.status === 401) {
    // Handle authentication error
  }
  
  throw error // Re-throw for component handling
}
```

## 🤖 AI Assistant Work History Rules

### Work History Documentation
- **ALWAYS** read the project history file at `d:\project\online-learning-system\online-learning-system-frontend\.trae\rules\project_history.md` before starting any work session
- **ALWAYS** document completed work with timestamps and detailed descriptions
- **ALWAYS** update the work history after completing significant tasks or features
- **ALWAYS** include context about what was changed, why it was changed, and any important decisions made

### Work History Structure
```markdown
## [YYYY-MM-DD HH:MM] - Task Title
**Context**: Brief description of the task or issue
**Changes Made**:
- Detailed list of changes
- Files modified
- New features added
- Bugs fixed

**Technical Decisions**:
- Important architectural decisions
- Library choices
- Pattern implementations

**Next Steps** (if applicable):
- Follow-up tasks
- Known issues to address
- Recommendations for future work

---
```

### Session Continuity Requirements
- **ALWAYS** start each work session by reviewing the latest work history entries
- **ALWAYS** understand the current state of the project before making changes
- **ALWAYS** maintain context awareness across different work sessions
- **NEVER** duplicate work that has already been completed
- **ALWAYS** build upon previous work rather than starting from scratch

### History File Management
- Keep work history entries in chronological order (newest first)
- Limit individual entries to essential information to maintain readability
- Archive old entries (older than 30 days) to a separate archive section
- Use clear, descriptive titles for each work session

## 🎯 Best Practices Summary

1. **Consistency**: Follow established patterns throughout the codebase
2. **Type Safety**: Use TypeScript strictly, avoid any types
3. **Performance**: Optimize for Core Web Vitals
4. **Accessibility**: Ensure WCAG 2.1 AA compliance
5. **Testing**: Maintain high test coverage
6. **Documentation**: Keep code and API docs updated
7. **Security**: Follow security best practices
8. **Maintainability**: Write clean, readable code
9. **Work History**: Maintain detailed work history for AI assistant continuity

---

**Note**: These rules should be followed by all team members and AI assistants. Any deviations should be discussed and approved by the team lead. AI assistants must read and update the work history file for every work session to ensure proper context and continuity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huynhduydong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/huynhduydong)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
