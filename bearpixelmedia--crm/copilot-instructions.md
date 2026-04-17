## crm

> - Always use TypeScript for all new files

# White Fox Studios CRM - Cursor Rules
# Optimized for Next.js 15, TypeScript, and Google Sheets Integration

## 🏗️ Project Structure & Architecture

- Always use TypeScript for all new files
- Follow Next.js 15 App Router conventions
- Use server components by default, client components only when needed
- Keep API routes in `app/api/` directory
- Use `lib/` for utility functions and shared logic
- Use `types/` for TypeScript type definitions
- Use `context/` for React context providers
- Use `hooks/` for custom React hooks

## 🎨 UI & Styling

- Use Tailwind CSS for all styling
- Use shadcn/ui components when available
- Follow the existing color scheme and design system
- Use CSS variables for theming (already defined in globals.css)
- Make components responsive by default
- Use semantic HTML elements
- Include proper accessibility attributes

## 📊 Data Management

- Always use the DataContext for client-side state management
- Use Google Sheets API for all data persistence
- Implement proper error handling for API calls
- Use loading states for async operations
- Validate data on both client and server side
- Use TypeScript interfaces for all data structures

## 🔐 Authentication & Security

- Use JWT tokens for authentication
- Hash passwords with bcrypt
- Implement role-based access control (admin, manager, agent)
- Validate user permissions on both client and server
- Use environment variables for sensitive data
- Never expose API keys in client-side code

## 🧪 Code Quality

- Use meaningful variable and function names
- Add JSDoc comments for complex functions
- Use proper TypeScript types (avoid `any`)
- Implement proper error boundaries
- Use React.memo for expensive components
- Follow ESLint and Prettier configurations

## 🚀 Performance

- Use Next.js Image component for images
- Implement proper loading states
- Use React.memo for expensive components
- Optimize bundle size with dynamic imports
- Use proper caching strategies for API calls
- Implement proper SEO meta tags

## 📱 Component Patterns

- Use composition over inheritance
- Keep components small and focused
- Use proper prop drilling or context as needed
- Implement proper form validation
- Use controlled components for forms
- Handle loading and error states gracefully

## 🔧 API Development

- Use proper HTTP status codes
- Implement consistent error responses
- Use TypeScript for API route handlers
- Validate request data
- Use proper CORS headers
- Implement rate limiting for production

## 📝 File Naming Conventions

- Use kebab-case for file names
- Use PascalCase for component names
- Use camelCase for functions and variables
- Use UPPER_SNAKE_CASE for constants
- Use descriptive names that indicate purpose

## 🎯 CRM-Specific Rules

- Always maintain data relationships (clients → projects → tasks)
- Use consistent ID formats (CL001, PRJ001, etc.)
- Implement proper audit trails for data changes
- Use proper date formatting (YYYY-MM-DD)
- Handle currency values consistently
- Implement proper search and filtering

## 🚨 Error Handling

- Use try-catch blocks for async operations
- Implement proper error boundaries
- Show user-friendly error messages
- Log errors appropriately
- Handle network failures gracefully
- Provide fallback UI for errors

## 📊 Testing Considerations

- Write testable code
- Use proper separation of concerns
- Mock external dependencies
- Test both success and error paths
- Use proper test data
- Implement integration tests for critical flows

## 🔄 State Management

- Use React Context for global state
- Use local state for component-specific data
- Avoid prop drilling when possible
- Use proper state update patterns
- Implement optimistic updates where appropriate
- Handle loading and error states consistently

## 🎨 UI/UX Guidelines

- Use consistent spacing and typography
- Implement proper focus states
- Use meaningful icons and labels
- Provide clear feedback for user actions
- Use proper color contrast
- Implement responsive design patterns

## 📈 Performance Monitoring

- Monitor bundle size
- Track API response times
- Monitor user interactions
- Implement proper analytics
- Track error rates
- Monitor loading performance

## 🔧 Development Workflow

- Use meaningful commit messages
- Implement proper branching strategy
- Use feature flags for new features
- Implement proper code review process
- Use automated testing where possible
- Maintain proper documentation

## 🚀 Deployment Considerations

- Use environment-specific configurations
- Implement proper build optimization
- Use CDN for static assets
- Implement proper caching strategies
- Monitor production performance
- Implement proper logging and monitoring 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bearpixelmedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
