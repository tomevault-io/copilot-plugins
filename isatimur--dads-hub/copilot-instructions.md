## dads-hub

> - feature-first organization

# Отец Молодец Project Coding Standards and Best Practices

# Project Architecture
architecture:
  structure:
    root:
      - src/
        - components/
          - {feature}/
            - ui/
            - hooks/
            - utils/
        - features/
        - lib/
        - types/
        - styles/
        - utils/
  patterns:
    - feature-first organization
    - atomic design principles
    - composition over inheritance
    - single responsibility

# Component Architecture
components:
  structure:
    - imports (grouped by type)
    - types/interfaces
    - constants
    - hooks
    - helper functions
    - component definition
    - exports
  principles:
    - small and focused
    - single responsibility
    - proper prop typing
    - error boundaries
    - loading states

# State Management
state:
  client:
    - React Query for server state
    - Context for global UI state
    - useState for local state
    - useReducer for complex state
  server:
    - Supabase real-time subscriptions
    - Optimistic updates
    - Proper error handling
    - Cache invalidation
  persistence:
    - Local storage for preferences
    - Supabase for data
    - Proper offline handling

# Performance
performance:
  optimization:
    - code splitting
    - lazy loading
    - proper caching
    - debounced operations
    - memoization
  assets:
    - image optimization
    - lazy loading
    - proper formats
    - CDN usage
  monitoring:
    - core web vitals
    - error tracking
    - performance metrics

# Security
security:
  authentication:
    - Supabase Auth
    - proper session handling
    - secure token storage
  authorization:
    - role-based access control
    - proper permissions
    - content policies
  data:
    - input sanitization
    - XSS prevention
    - CORS configuration
    - rate limiting

# Real-time Features
realtime:
  supabase:
    - proper channel management
    - optimistic updates
    - error handling
    - reconnection logic
  patterns:
    - presence
    - broadcast
    - direct messages
    - typing indicators

# Testing
testing:
  unit:
    - components
    - hooks
    - utilities
  integration:
    - user flows
    - api interactions
  e2e:
    - critical paths
    - user journeys
  coverage:
    minimum: 80%

# Styling
styling:
  methodology:
    - utility-first with Tailwind
    - component-based design
    - consistent spacing
    - responsive design
  themes:
    - light/dark mode
    - consistent colors
    - proper contrast
    - accessible design
  components:
    - shadcn/ui base
    - consistent animations
    - proper loading states
    - error states

# Documentation
documentation:
  code:
    - JSDoc comments
    - type definitions
    - usage examples
    - edge cases
  api:
    - endpoints
    - parameters
    - responses
    - error codes
  project:
    - README
    - contributing guide
    - deployment guide
    - architecture decisions

# Development Workflow
workflow:
  git:
    branches:
      format: "{type}/{description}"
      types:
        - feat
        - fix
        - refactor
        - docs
        - style
        - test
    commits:
      format: "type(scope): description"
      scope: optional
  review:
    - code review checklist
    - performance impact
    - security considerations
    - accessibility requirements

# Error Handling
errors:
  client:
    - proper error boundaries
    - user-friendly messages
    - retry mechanisms
    - fallback UI
  server:
    - proper status codes
    - detailed error messages
    - error logging
    - recovery strategies

# Accessibility
accessibility:
  requirements:
    - WCAG 2.1 compliance
    - keyboard navigation
    - screen reader support
    - proper contrast
    - proper semantics
  testing:
    - automated checks
    - manual testing
    - screen reader testing
    - keyboard navigation testing

# Monitoring
monitoring:
  metrics:
    - error rates
    - performance
    - user engagement
    - api usage
  logging:
    - error tracking
    - user actions
    - performance metrics
    - security events

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isatimur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
