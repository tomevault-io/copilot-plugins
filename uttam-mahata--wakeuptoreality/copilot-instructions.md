## wakeuptoreality

> This is a wake-up call system using Twilio Voice API, NestJS backend, and React frontend.

# Wake Up To Reality - Cursor AI Rules

## Project Overview
This is a wake-up call system using Twilio Voice API, NestJS backend, and React frontend.

## Tech Stack
- **Backend**: NestJS, TypeScript, Prisma ORM, PostgreSQL
- **Frontend**: React, TypeScript, TailwindCSS, React Query
- **Voice Service**: Twilio Programmable Voice API
- **Scheduler**: Node-cron
- **Deployment**: Docker containers

## Code Style & Standards

### General
- Use TypeScript strict mode
- Follow SOLID principles
- Write self-documenting code with meaningful names
- Prefer functional programming patterns where appropriate
- Use async/await over promises chains
- Handle errors gracefully with proper error messages

### NestJS Backend
- Use dependency injection for all services
- Implement proper DTOs with class-validator decorators
- Use Guards for authentication/authorization
- Implement interceptors for logging and transformation
- Follow module-per-feature architecture
- Use configuration service for environment variables
- Document all API endpoints with Swagger decorators
- Use Prisma for database operations with proper transactions

### React Frontend
- Use functional components with hooks
- Implement proper TypeScript interfaces for props
- Use React Query for server state management
- Use Context API or Zustand for client state
- Implement proper error boundaries
- Use TailwindCSS for styling (no inline styles)
- Follow atomic design principles (atoms, molecules, organisms)
- Implement proper loading and error states
- Use React Hook Form for form handling

### API Design
- Follow RESTful conventions
- Use proper HTTP status codes
- Implement pagination for list endpoints
- Use camelCase for JSON properties
- Implement proper request validation
- Return consistent error response format
- Use JWT for authentication
- Implement rate limiting

### Database
- Use migrations for schema changes
- Index frequently queried fields
- Use transactions for multi-step operations
- Implement soft deletes where appropriate
- Use UUID for primary keys
- Store timestamps in UTC

### Security
- Never commit secrets or API keys
- Use environment variables for sensitive data
- Implement CORS properly
- Sanitize user inputs
- Use helmet for security headers
- Implement request rate limiting
- Hash passwords with bcrypt
- Validate JWT tokens properly

### Testing
- Write unit tests for business logic
- Write integration tests for API endpoints
- Mock external services (Twilio)
- Aim for 80%+ code coverage
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)

### File Structure
```
backend/
├── src/
│   ├── modules/          # Feature modules
│   ├── common/           # Shared utilities
│   ├── config/           # Configuration
│   └── prisma/           # Database schema & migrations

frontend/
├── src/
│   ├── components/       # Reusable components
│   ├── pages/           # Page components
│   ├── hooks/           # Custom hooks
│   ├── services/        # API services
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
```

### Git Conventions
- Use conventional commits (feat:, fix:, docs:, etc.)
- Write clear commit messages
- Keep commits atomic and focused
- Create feature branches (feature/*, fix/*)
- Squash commits before merging to main

### Documentation
- Document complex logic with comments
- Keep README up to date
- Document environment variables
- Write API documentation with examples
- Include setup instructions

### Performance
- Implement caching where appropriate
- Use database indexes strategically
- Optimize bundle size (code splitting)
- Lazy load routes and heavy components
- Implement proper pagination
- Use connection pooling for database

### Error Handling
- Use custom exception filters in NestJS
- Return user-friendly error messages
- Log errors with proper context
- Implement retry logic for external APIs
- Use circuit breaker pattern for Twilio calls

## Twilio Integration Guidelines
- Use Twilio SDK, not raw HTTP calls
- Implement proper error handling for Twilio API failures
- Store call logs in database
- Implement retry mechanism with exponential backoff
- Use webhooks for call status updates
- Test with Twilio test credentials in development
- Implement call status tracking (initiated, ringing, answered, completed, failed)

## Development Workflow
1. Create feature branch from main
2. Implement feature with tests
3. Ensure all tests pass
4. Update documentation if needed
5. Create pull request with clear description
6. Review and merge after approval

## Environment Variables
- Use .env files (never commit them)
- Provide .env.example files
- Validate environment variables on startup
- Use different configs for dev/staging/prod

## Code Review Checklist
- [ ] Code follows style guidelines
- [ ] Tests are written and passing
- [ ] No console.logs in production code
- [ ] Error handling is implemented
- [ ] Documentation is updated
- [ ] No hardcoded values
- [ ] TypeScript types are properly defined
- [ ] Security best practices followed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Uttam-Mahata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
