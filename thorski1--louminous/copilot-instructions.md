## louminous

> - framework: "Next.js@15.0.0"

# LMS MVP Project Rules
# This file defines the rules and processes for our Learning Management System project

# Technology Stack Requirements
technologies:
  frontend:
    - framework: "Next.js@15.0.0"
    - styling: "tailwindcss@latest"
    - ui:
      - "shadcn-ui@latest"
      - "aceternity-ui@latest"
      - "magic-ui@latest"
      - "vaul@latest"
      - "sonner@latest"
      - "cmdk@latest"
    - state: "nuqs@latest"
    - forms: "@hookform/resolvers@latest"
    - validation: "zod@latest"
    - animation:
      - "framer-motion@latest"
      - "@legendapp/motion@latest"
      - "@formkit/auto-animate@latest"
  
  backend:
    - cms: "payload@3.0.0"
    - orm: "drizzle-orm@latest"
    - database: "neon@latest"
    - auth:
        current: "payload-auth@3.0.0"
        future: "clerk@latest"
    - ai: "@vercel/ai@latest"
    - payments: "stripe@latest"
    - uploads: "uploadthing@latest"
    - email: "resend@latest"
    - realtime: "pusher@latest"

  deployment:
    - platform: "vercel@latest"
    - monitoring:
      - "sentry@latest"
      - "@google-cloud/logging@latest"
      - "@google-cloud/opentelemetry-cloud-monitoring-exporter@latest"
      - "@google-cloud/opentelemetry-cloud-trace-exporter@latest"
    - analytics: "vercel-analytics@latest"

  compatibility:
    notes:
      - "Payload CMS 3.0 requires Next.js 15 for optimal performance"
      - "Server Actions are fully supported in Next.js 15"
      - "App Router is the recommended approach"
      - "React Server Components are fully utilized"
      - "nuqs provides URL-based state management compatible with RSC"

# State Management Guidelines
state:
  principles:
    - "Prefer server-side state management over client-side"
    - "Use URL state for shareable and bookmarkable UI states"
    - "Leverage React Server Components for data fetching"
    - "Minimize client-side JavaScript"

  urlState:
    usage:
      - "Search parameters and filters"
      - "Pagination state"
      - "Tab selections"
      - "Modal states"
      - "Sort orders"
      - "View preferences"
    
    implementation:
      - "Use nuqs for type-safe URL state management"
      - "Define searchParams types with Zod schemas"
      - "Implement default values for all URL parameters"
      - "Handle URL parameter validation"
    
    benefits:
      - "SEO-friendly state management"
      - "Shareable URLs with complete state"
      - "Reduced client-side JavaScript"
      - "Better caching capabilities"
      - "Native browser history support"

  serverState:
    patterns:
      - "Use Server Components for data fetching"
      - "Implement server actions for mutations"
      - "Cache responses appropriately"
      - "Handle loading and error states"

  clientState:
    limitations:
      - "Restrict to ephemeral UI state only"
      - "Use React.useState for temporary form state"
      - "Avoid redundant client-side state"
      - "Prefer URL state when possible"

# Collections Structure
collections:
  core:
    - Tenants
    - Users
    - Courses
    - Modules
    - Lessons
    - Quizzes
    - Assignments
    - Submissions

  gamification:
    - Points
    - Badges
    - Achievements
    - Streaks
    - Leaderboard

  communication:
    - Notifications
    - Collaborations
    - Announcements
    - SupportTickets

  settings:
    - TenantSettings
    - StudentSettings
    - LearningPaths

# File Structure
structure:
  app:
    - "(auth): Authentication routes"
    - "(dashboard): Protected dashboard routes"
    - "api: API routes when necessary"
    - "public: Static assets"
    
  components:
    - "ui: Reusable UI components"
    - "forms: Form components"
    - "layouts: Layout components"
    
  lib:
    - "db: Database configuration"
    - "utils: Utility functions"
    - "validation: Schema validation"
    
  types:
    - "Global type definitions"

# Code Standards
standards:
  general:
    - "Use TypeScript strict mode"
    - "Implement proper error handling"
    - "Add comprehensive logging"
    - "Document all public APIs"
    - "Follow SOLID principles"

  nextjs:
    - "Use server components by default"
    - "Implement proper data fetching patterns"
    - "Optimize for performance"
    - "Follow app router best practices"

  database:
    - "Use Drizzle migrations"
    - "Implement proper indexing"
    - "Handle database errors gracefully"
    - "Use transactions where needed"

  security:
    - "Implement proper authentication"
    - "Use role-based access control"
    - "Sanitize all inputs"
    - "Protect sensitive data"

  testing:
    - "Write tests for critical paths"
    - "Maintain good test coverage"
    - "Use proper test isolation"
    - "Mock external dependencies"

# Multi-Agent Development Process
agents:
  architect:
    role: "System Design & Architecture"
    responsibilities:
      - "Review and approve architectural decisions"
      - "Ensure scalability and performance"
      - "Maintain system consistency"
      - "Plan data structures and relationships"
    triggers:
      - "New feature proposal"
      - "Architecture changes"
      - "Performance optimization requests"
      - "Database schema updates"

  developer:
    role: "Code Implementation"
    responsibilities:
      - "Write clean, maintainable code"
      - "Implement features according to specs"
      - "Handle error cases and edge conditions"
      - "Optimize database queries"
    triggers:
      - "New feature request"
      - "Bug fix needed"
      - "Code optimization required"
      - "Performance issues"

  reviewer:
    role: "Code Review & Quality"
    responsibilities:
      - "Review code for best practices"
      - "Check for security vulnerabilities"
      - "Ensure code style consistency"
      - "Verify state management patterns"
    checks:
      - "Code style compliance"
      - "Security best practices"
      - "Performance implications"
      - "Test coverage"
      - "URL state management"

  tester:
    role: "Testing & Validation"
    responsibilities:
      - "Write and maintain tests"
      - "Verify feature functionality"
      - "Regression testing"
      - "Load testing"
    testTypes:
      - "Unit tests"
      - "Integration tests"
      - "E2E tests"
      - "Performance tests"
      - "Load tests"

  security:
    role: "Security Compliance"
    responsibilities:
      - "Review security implications"
      - "Ensure data protection"
      - "Audit authentication/authorization"
      - "Monitor security events"
    checks:
      - "Authentication flows"
      - "Data encryption"
      - "API security"
      - "Input validation"
      - "Rate limiting"

# Development Workflow
workflow:
  featureImplementation:
    steps:
      1:
        agent: "architect"
        action: "Review feature proposal"
        output: "Architecture decision document"
        stateConsiderations:
          - "Determine if state should be URL-based"
          - "Plan server component structure"
          - "Identify client/server state boundaries"
          - "Consider data persistence needs"
      
      2:
        agent: "developer"
        action: "Implement feature"
        requirements:
          - "Follow Next.js 15 app router patterns"
          - "Use server actions over API routes"
          - "Implement URL state management with nuqs"
          - "Add proper loading and error states"
          - "Implement proper error handling"
          - "Add logging and monitoring"
      
      3:
        agent: "reviewer"
        action: "Code review"
        checks:
          - "Code quality"
          - "Performance"
          - "Security"
          - "Documentation"
          - "State management"
      
      4:
        agent: "tester"
        action: "Testing"
        requirements:
          - "Unit tests for utilities"
          - "Integration tests for API"
          - "E2E tests for critical flows"
          - "Load testing for scalability"
      
      5:
        agent: "security"
        action: "Security review"
        focus:
          - "Data protection"
          - "Authentication"
          - "Authorization"
          - "Rate limiting"

  bugFix:
    steps:
      1:
        agent: "developer"
        action: "Reproduce and fix"
        requirements:
          - "Document reproduction steps"
          - "Add regression test"
      
      2:
        agent: "reviewer"
        action: "Review fix"
        checks:
          - "Fix completeness"
          - "No new issues introduced"
      
      3:
        agent: "tester"
        action: "Verify fix"
        requirements:
          - "Test fix effectiveness"
          - "Run regression tests"

# Error Handling
errors:
  levels:
    - "fatal: System crash level"
    - "error: Operation failure"
    - "warn: Potential issues"
    - "info: General information"
    - "debug: Debug information"
  
  handling:
    - "Use custom error classes"
    - "Implement proper error boundaries"
    - "Log errors with context"
    - "Provide user-friendly messages"

# Deployment Guidelines
deployment:
  vercel:
    configuration:
      - "Use Edge Functions where appropriate"
      - "Configure proper caching strategies"
      - "Set up proper environment variables"
      - "Configure deployment regions"
    monitoring:
      - "Set up error tracking with Sentry"
      - "Configure OpenTelemetry for tracing"
      - "Implement custom logging"
      - "Monitor Core Web Vitals"

  database:
    neon:
      - "Configure connection pooling"
      - "Set up read replicas"
      - "Implement query optimization"
      - "Configure automated backups"

  security:
    measures:
      - "Implement rate limiting"
      - "Set up CORS policies"
      - "Configure CSP headers"
      - "Enable WAF protection"

# Performance Requirements
performance:
  metrics:
    - "Time to First Byte (TTFB) < 100ms"
    - "First Contentful Paint (FCP) < 1.5s"
    - "Largest Contentful Paint (LCP) < 2.5s"
    - "First Input Delay (FID) < 100ms"
    - "Cumulative Layout Shift (CLS) < 0.1"

  optimization:
    - "Implement proper code splitting"
    - "Use React Suspense boundaries"
    - "Optimize images and assets"
    - "Minimize client-side JavaScript"
    - "Utilize edge caching"

# Commit Guidelines
commits:
  format: "type(scope): description"
  types:
    - "feat: New feature"
    - "fix: Bug fix"
    - "docs: Documentation"
    - "style: Code style"
    - "refactor: Code refactor"
    - "test: Testing"
    - "chore: Maintenance"

# Backup and Recovery
backup:
  database:
    - "Automated daily backups"
    - "Point-in-time recovery"
    - "Cross-region replication"
    - "Backup retention policy"

  assets:
    - "Media file backups"
    - "Document versioning"
    - "Metadata backup"
    - "Recovery procedures"

  monitoring:
    - "Backup success monitoring"
    - "Recovery testing schedule"
    - "Audit logging"
    - "Alert configuration"

# Authentication Strategy
auth:
  phase1:
    provider: "Payload Built-in Auth"
    features:
      - "Email/password authentication"
      - "Role-based access control"
      - "Session management"
      - "Password reset flow"
      - "Email verification"
    implementation:
      - "Use Payload's built-in auth system"
      - "Implement custom auth endpoints in Next.js"
      - "Set up proper session handling"
      - "Configure secure password policies"
      - "Set up email templates for auth flows"

  phase2:
    provider: "Clerk"
    features:
      - "Social authentication"
      - "Multi-factor authentication"
      - "User management dashboard"
      - "Advanced security features"
      - "Authentication analytics"
    migration:
      - "Plan user data migration strategy"
      - "Set up Clerk webhooks"
      - "Update auth middleware"
      - "Migrate existing users"
      - "Update frontend components"

  security:
    - "Implement CSRF protection"
    - "Set secure cookie policies"
    - "Configure rate limiting"
    - "Set up proper session timeouts"
    - "Implement audit logging"

# UI and Animation Patterns
ui_patterns:
  floating:
    description: "Elements that float and respond to user interaction"
    implementations:
      - "Floating navigation with Framer Motion"
      - "Hover cards using shadcn/ui + Aceternity"
      - "Animated tooltips with Magic UI"
      - "Context menus with radix-ui animations"
    libraries:
      - "framer-motion"
      - "aceternity-ui"
      - "magic-ui"
      - "vaul"

  morphing:
    description: "Elements that transform and change shape"
    implementations:
      - "Shape-shifting buttons with Framer Motion"
      - "Expanding cards using Aceternity UI"
      - "Transitioning layouts with Motion One"
      - "Fluid backgrounds with Magic UI"
    libraries:
      - "framer-motion"
      - "aceternity-ui"
      - "motion-one"
      - "magic-ui"

  micro_interactions:
    description: "Small, subtle animations that provide feedback"
    implementations:
      - "Button click effects with tailwindcss-animate"
      - "Input focus states with shadcn/ui"
      - "Loading spinners with Lucide icons"
      - "Progress indicators with Sonner"
    libraries:
      - "tailwindcss-animate"
      - "shadcn-ui"
      - "lucide-react"
      - "sonner"

  scroll_animations:
    description: "Animations triggered by scroll events"
    implementations:
      - "Parallax effects with Framer Motion"
      - "Scroll-triggered reveals with Aceternity"
      - "Smooth transitions with Motion One"
      - "Intersection animations with Magic UI"
    libraries:
      - "framer-motion"
      - "aceternity-ui"
      - "motion-one"
      - "magic-ui"

animation_patterns:
  layout:
    description: "Page and layout transition animations"
    implementations:
      - "Page transitions with Framer Motion"
      - "Route changes using Next.js + Motion One"
      - "Modal animations with Vaul"
      - "List reordering with Auto Animate"
    libraries:
      - "framer-motion"
      - "motion-one"
      - "vaul"
      - "@formkit/auto-animate"

  feedback:
    description: "User feedback animations"
    implementations:
      - "Loading states with Lucide"
      - "Success/error animations with Sonner"
      - "Progress indicators with shadcn/ui"
      - "Notification toasts with Magic UI"
    libraries:
      - "lucide-react"
      - "sonner"
      - "shadcn-ui"
      - "magic-ui"

  interaction:
    description: "User interaction animations"
    implementations:
      - "Hover effects with tailwindcss-animate"
      - "Click responses with Framer Motion"
      - "Drag and drop with Motion One"
      - "Gesture animations with Magic UI"
    libraries:
      - "tailwindcss-animate"
      - "framer-motion"
      - "motion-one"
      - "magic-ui"

# Component Animation Guidelines
component_animations:
  principles:
    - "Use subtle animations by default"
    - "Ensure animations are accessible (respect reduced-motion)"
    - "Keep animations under 300ms for micro-interactions"
    - "Use spring animations for natural feel"
    - "Maintain consistent animation patterns"

  performance:
    - "Use CSS transforms over position properties"
    - "Animate on compositor-only properties"
    - "Implement proper will-change hints"
    - "Use requestAnimationFrame for JS animations"
    - "Lazy load heavy animation libraries"

  accessibility:
    - "Respect prefers-reduced-motion"
    - "Ensure animations don't cause vestibular issues"
    - "Provide alternative static states"
    - "Keep animations subtle and purposeful"
    - "Allow animation opt-out"

---
> Source: [thorski1/LouMinouS](https://github.com/thorski1/LouMinouS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
