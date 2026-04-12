## doc-extract

> Frontend development guidelines


# Document Extraction Platform - Development Rules

## General Rules
- Don't jump to conclusions
- Don't make assumptions
- Ask clarifying questions

## Solving Problems
- Explain the error in simple terms
- Suggest 3 possible best solutions

## Development Approach
- Follow the user's requirements carefully
- Ask clarifying questions
- First think step-by-step - describe your plan for what to build in pseudocode, written out in great detail
- Confirm, then write the code!
- Focus on easy and readable code, over being performant
- Fully implement all requested functionality
- Leave NO todos, placeholder or missing pieces
- Ensure code is complete! Verify thoroughly finalised
- Include all required imports, and ensure proper naming of key components
- If you think that there might not be a correct answer, say so
- If you don't know the answer, say so, instead of guessing
- Use early returns whenever possible to make the code more readable

## Conflict Resolution
- When user request a change that you believe to break the rule, mention the rule and ask the user to verify the request
- If possible, put a comment where it makes most sense, why are we breaking cursor rules

## Code Organisation
- Keep code organized by feature or domain
- Use clear and consistent naming conventions
- Follow the established project structure
- Document complex logic or business rules
- Use TypeScript for type safety where possible

## Git Workflow
- Use conventional commits
- Use conventional branch names
- git status
- git add <changed files>
- git commit -m "what changed"
- git push

# Frontend Development Rules

## 🚨 CRITICAL: Investigation First
**ALWAYS investigate existing code, patterns, and implementations before creating anything new.**
- Check existing components, styled components, and theme values
- Look for similar functionality that already exists
- Verify database schema and API endpoints before creating migrations
- Never create duplicate functionality or files
- Ask clarifying questions when unsure about requirements

## 🎨 Styling and Theme Guidelines

### Use Theme System
- **ALWAYS** use theme colors from `frontend/src/styles/theme.ts`
- **NEVER** use hardcoded colors like `#3b82f6`, `#1f2937`, `#6b7280`
- **Primary color**: `theme.colors.primary` (#2563eb - blue)
- **Text colors**: `theme.colors.text.primary`, `theme.colors.text.secondary`
- **Spacing**: Use `theme.spacing.sm`, `theme.spacing.md`, etc.
- **Typography**: Use `theme.typography.weights.medium`, `theme.typography.sizes.sm`

### Styled Components
- **PREFER** styled-components over inline styles
- **USE** existing styled components instead of creating new ones
- **AVOID** inline styles with hardcoded values
- **FOLLOW** established component patterns in the codebase

### Button and UI Components
- **USE** the existing `Button` component with proper variants (`primary`, `secondary`, `outline`, `ghost`, `danger`)
- **FOLLOW** the established color scheme (blue primary, not orange or other colors)
- **USE** `Button` from `@/components/ui/Button` instead of custom styled buttons

## 🧩 Component Architecture

### Memoization and Performance
- **ALWAYS** wrap functions in `useCallback` when used in dependency arrays
- **ALWAYS** wrap context values in `useMemo` to prevent unnecessary re-renders
- **ENSURE** stable function references to prevent infinite re-renders
- **USE** proper dependency arrays in `useCallback` and `useMemo`

### Existing Components
- **CHECK** for existing styled components before creating new ones
- **REUSE** existing patterns and components
- **AVOID** duplicating functionality that already exists
- **FOLLOW** established component naming conventions

### Error Handling
- **USE** proper error boundaries and error states
- **PROVIDE** meaningful error messages to users
- **HANDLE** loading states for async operations
- **IMPLEMENT** proper fallback UI for error scenarios

## 📊 State Management

### Loading and Error States
- **ALWAYS** provide loading indicators for async operations
- **HANDLE** error states gracefully with user-friendly messages
- **USE** proper error handling patterns (try/catch, error boundaries)
- **IMPLEMENT** retry logic for failed operations

### Empty States
- **PROVIDE** clean, professional empty states when no data exists
- **USE** consistent empty state patterns across the application
- **INCLUDE** clear call-to-action buttons in empty states
- **AVOID** showing fake/mock data in production

### API State Management
- **HANDLE** concurrent API calls properly
- **IMPLEMENT** proper token refresh logic with race condition prevention
- **USE** proper error handling for authentication failures
- **PROVIDE** user feedback for API operations

## 🔐 Authentication and Security

### Token Management
- **IMPLEMENT** proper token refresh with race condition prevention
- **HANDLE** 401 errors gracefully with automatic token refresh
- **PREVENT** multiple concurrent refresh attempts
- **PROVIDE** proper logout handling on authentication failures

### Error Handling
- **USE** proper logging instead of print statements
- **IMPLEMENT** `logger.exception()` for better error diagnostics
- **HANDLE** authentication errors gracefully
- **PROVIDE** proper user feedback for security issues

## 🎯 User Experience

### Table and Data Display
- **IMPLEMENT** proper text overflow handling with ellipsis
- **USE** consistent table styling and layouts
- **PROVIDE** proper loading states for data tables
- **HANDLE** empty states professionally

### Form Handling
- **USE** proper form validation and error handling
- **PROVIDE** clear feedback for form submissions
- **HANDLE** loading states during form operations
- **IMPLEMENT** proper accessibility features

### Timezone Handling
- **HANDLE** timezone conversions properly for datetime inputs
- **USE** local timezone for user inputs
- **CONVERT** to UTC for backend submission
- **PROVIDE** proper timezone-aware date/time handling

## 🚀 Performance and Optimization

### React Patterns
- **USE** `useCallback` for functions passed as props or in dependency arrays
- **USE** `useMemo` for expensive calculations and context values
- **AVOID** creating new objects/functions on every render
- **IMPLEMENT** proper cleanup in useEffect hooks

### API Optimization
- **IMPLEMENT** proper caching for API responses
- **USE** loading states to prevent multiple concurrent requests
- **HANDLE** race conditions in async operations
- **PROVIDE** proper error recovery mechanisms

## 📱 Responsive Design

### Layout and Spacing
- **USE** consistent spacing from the theme system
- **IMPLEMENT** responsive layouts that work on all screen sizes
- **FOLLOW** established layout patterns
- **USE** proper flexbox and grid layouts

### Accessibility
- **PROVIDE** proper ARIA labels and roles
- **USE** semantic HTML elements
- **IMPLEMENT** keyboard navigation support
- **PROVIDE** proper focus management

## 🔧 Development Practices

### Code Quality
- **FOLLOW** TypeScript best practices
- **USE** proper type definitions
- **IMPLEMENT** proper error handling
- **WRITE** clean, readable, and maintainable code

### Testing
- **WRITE** tests for critical functionality
- **TEST** error scenarios and edge cases
- **VERIFY** user interactions and workflows
- **ENSURE** proper test coverage

### Documentation
- **DOCUMENT** complex logic and business rules
- **PROVIDE** clear comments for non-obvious code
- **MAINTAIN** up-to-date documentation
- **EXPLAIN** design decisions and trade-offs

## 🚫 What NOT to Do

### Avoid These Patterns
- **DON'T** use hardcoded colors or magic numbers
- **DON'T** create duplicate functionality
- **DON'T** use inline styles with hardcoded values
- **DON'T** ignore existing patterns and components
- **DON'T** create migrations without checking existing schema
- **DON'T** use print statements for logging
- **DON'T** show fake/mock data in production
- **DON'T** ignore error handling and loading states
- **DON'T** create unstable function references
- **DON'T** ignore timezone handling in datetime inputs

## ✅ Quality Checklist

Before submitting any code, ensure:
- [ ] Investigated existing code and patterns
- [ ] Used theme colors and spacing
- [ ] Used existing styled components
- [ ] Implemented proper error handling
- [ ] Added loading states for async operations
- [ ] Used proper memoization (useCallback/useMemo)
- [ ] Handled empty states professionally
- [ ] Used proper TypeScript types
- [ ] Followed established naming conventions
- [ ] Tested the implementation thoroughly

## 🎯 Success Criteria

Code should be:
- **Consistent** with existing patterns
- **Maintainable** and easy to understand
- **Performant** with proper optimization
- **Accessible** and user-friendly
- **Robust** with proper error handling
- **Professional** in appearance and behavior

## 🌐 API Services - Domain-Driven Design

### Core Principles
- **MANDATORY**: All API calls must be organized by business domain
- Each domain should have its own dedicated service class
- Domains should be separated from the monolithic `services/api.ts` file
- Each domain service should handle all operations for that specific business entity

### Directory Structure
```
src/services/
├── api/
│   ├── base/
│   │   ├── BaseApiClient.ts
│   │   └── types/
│   │       └── common.ts
│   ├── auth/
│   │   ├── AuthService.ts
│   │   ├── AuthService.test.ts
│   │   └── types/
│   │       └── auth.ts
│   ├── documents/
│   │   ├── DocumentService.ts
│   │   ├── DocumentService.test.ts
│   │   └── types/
│   │       └── documents.ts
│   ├── templates/
│   │   ├── TemplateService.ts
│   │   ├── TemplateService.test.ts
│   │   └── types/
│   │       └── templates.ts
│   ├── extractions/
│   │   ├── ExtractionService.ts
│   │   ├── ExtractionService.test.ts
│   │   └── types/
│   │       └── extractions.ts
│   ├── tenants/
│   │   ├── TenantService.ts
│   │   ├── TenantService.test.ts
│   │   └── types/
│   │       └── tenants.ts
│   ├── jobs/
│   │   ├── JobService.ts
│   │   ├── JobService.test.ts
│   │   └── types/
│   │       └── jobs.ts
│   ├── language/
│   │   ├── LanguageService.ts
│   │   ├── LanguageService.test.ts
│   │   └── types/
│   │       └── language.ts
│   ├── categories/
│   │   ├── CategoryService.ts
│   │   ├── CategoryService.test.ts
│   │   └── types/
│   │       └── categories.ts
│   └── health/
│       ├── HealthService.ts
│       ├── HealthService.test.ts
│       └── types/
│           └── health.ts
└── index.ts
```

### Service Class Design
```typescript
// Base API Client
export abstract class BaseApiClient {
  protected client: AxiosInstance;
  protected authToken: string | null = null;

  constructor(client: AxiosInstance) {
    this.client = client;
  }

  protected async request<T>(config: AxiosRequestConfig): Promise<T> {
    // Common request logic with auth, error handling, etc.
  }

  setAuthToken(token: string | null): void {
    this.authToken = token;
  }
}

// Domain Service Pattern
export class DocumentService extends BaseApiClient {
  constructor(client: AxiosInstance) {
    super(client);
  }

  async getDocuments(params: DocumentListParams): Promise<DocumentListResponse> {
    return this.request<DocumentListResponse>({
      method: 'GET',
      url: '/api/documents',
      params
    });
  }

  // All document-related operations...
}
```

### Testing Requirements
- **MANDATORY**: Every service class must have a corresponding test file
- Test files must be co-located with service files
- Tests should cover:
  - All public methods
  - Error handling scenarios
  - Authentication flows
  - Request/response transformations

### Migration Rules
- **When modifying any API method in `services/api.ts`:**
  1. Identify the domain (auth, documents, templates, etc.)
  2. Create the domain service directory if it doesn't exist
  3. Move the method to the appropriate domain service
  4. Create corresponding types in the domain types file
  5. Write tests for the moved method
  6. Update imports throughout the application
  7. Remove the method from `services/api.ts`

### Quality Gates
- **No new methods in `services/api.ts`** - All new API methods must go to domain services
- **All domain services must have tests** - No service without corresponding test file
- **Type safety required** - All service methods must have proper TypeScript types
- **Error handling mandatory** - All services must handle errors consistently

## 🧪 API Testing Requirements

### Test Coverage Requirements
**Every public method must be tested:**
- ✅ Happy path scenarios
- ✅ Error handling scenarios
- ✅ Edge cases and boundary conditions
- ✅ Authentication flows
- ✅ Request/response transformations
- ✅ Parameter validation

### Coverage Requirements
- **Minimum 90% line coverage** for all service classes
- **100% coverage** for critical authentication and error handling code
- **All public methods** must have tests
- **All error paths** must be tested

### Test Structure Template
```typescript
// {Domain}Service.test.ts
import { {Domain}Service } from './{Domain}Service';
import { mockAxiosInstance } from '../../__mocks__/axios';

describe('{Domain}Service', () => {
  let {domain}Service: {Domain}Service;

  beforeEach(() => {
    {domain}Service = new {Domain}Service(mockAxiosInstance);
  });

  describe('methodName', () => {
    it('should handle success case', async () => {
      // Arrange, Act, Assert
    });

    it('should handle errors gracefully', async () => {
      // Error handling test
    });
  });
});
```

## 🎨 Styled Components Rules

### Always Use Styled Components
- **MANDATORY**: Always use styled-components for styling
- Never use inline styles in components or pages
- All styles must be defined in styled component files

### Component File Organization
- **Each component must have its own folder structure:**
  ```
  component-name/
    ├── component-name.tsx
    └── component-name.styled.ts
  ```

### Separate Styled Components Files
- **Styled components must be in separate files** with `.styled.ts` extension
- Example: `Table.tsx` should have corresponding `Table.styled.ts`
- When working on existing components with inline styles, **refactor immediately** to move styles to separate `.styled.ts` file

### Component Reuse
- **ALWAYS check for existing components** before creating new ones
- Reuse existing components from `frontend/src/components`
- Check shared components in `frontend/src/components/common`
- Verify table components in `frontend/src/components/table`

### Theme Usage
- **Avoid primitive values** in styled component files
- **ALWAYS use theme values** from `frontend/src/styles/theme.ts`
- Use theme colors, spacing, typography, shadows, etc.

### Import Pattern
- Import styled components from the corresponding `.styled.ts` file:
  ```typescript
  import { 
    ButtonContainer,
    ButtonText,
    IconWrapper 
  } from './Button.styled';
  ```

### Refactoring Existing Components
- When modifying existing components that have inline styles:
  1. Create corresponding `.styled.ts` file
  2. Move all styles to the styled file
  3. Update component to use styled components
  4. Ensure all primitive values use theme values

### Enforcement
- **No exceptions** - these rules apply to ALL frontend components
- **Immediate refactoring** required when violations are found
- **Code review** must verify compliance with these styling guidelines

## Backend Development Rules
Backend-specific rules are defined in `.cursor/rules/backend-development.mdc` and apply to all Python/FastAPI code in the `backend/` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ieazie)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ieazie)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
