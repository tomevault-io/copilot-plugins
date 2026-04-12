## heysf

> Before implementing any solution, you MUST:

# Cursor Agent Stylistic Guidelines

## Core Decision-Making Process

### 1. ALWAYS REASON THROUGH ALTERNATIVES
Before implementing any solution, you MUST:
- Present at least 2 different approaches to solve the problem
- Briefly explain the pros/cons of each approach
- Clearly state which approach you're choosing and why
- Ask the human for confirmation if the decision significantly impacts architecture
- Always ask human clarifying questions if you have open questions that can improve the projec or your decision-making

Example format:

I see two main approaches for [problem]:
Approach 1: [Brief description]

Pros: [list]
Cons: [list]

Approach 2: [Brief description]

Pros: [list]
Cons: [list]

I recommend Approach X because [specific reasoning]. Proceeding with implementation.

### 2. SIMPLICITY FIRST PRINCIPLE
Before writing any code, ask yourself:
- "Is there a simpler way to solve this?"
- "Am I adding unnecessary complexity?"
- "Can I solve this with fewer dependencies/abstractions?"
- "Would a junior developer understand this code in 6 months?"

**Prefer:**
- Built-in Python libraries over external ones
- Simple functions over complex classes
- Direct database queries over heavy ORMs when simple
- Explicit code over clever abstractions
- Readable variable names over short ones

## Code Quality Standards

### Python Best Practices
- Follow PEP 8 style guidelines
- Use type hints for all function parameters and return values
- Write docstrings for all functions, classes, and modules
- Use meaningful variable and function names
- Keep functions under 20 lines when possible
- Use list comprehensions only when they improve readability

### Error Handling
- Use specific exception types, not bare `except:`
- Log errors with context, not just the exception
- Fail fast - validate inputs early
- Provide helpful error messages for users

### FastAPI Specific
- Use Pydantic models for request/response validation
- Group related endpoints in routers
- Use dependency injection for database connections
- Add response models to all endpoints
- Include proper HTTP status codes

### Database & SQLAlchemy
- Use descriptive table and column names
- Always define relationships explicitly
- Use database constraints where appropriate
- Write migrations for any schema changes
- Query only the fields you need

### Frontend (Next.js/React)
- Use functional components with hooks
- Keep components small and focused
- Use TypeScript interfaces for all props
- Handle loading and error states explicitly
- Use semantic HTML elements

## File Organization & Structure

### Backend Structure
backend/
├── app/
│   ├── init.py
│   ├── main.py              # FastAPI app setup
│   ├── config.py            # Environment configuration
│   ├── database.py          # Database connection
│   ├── models/              # SQLAlchemy models
│   ├── routers/             # API route handlers
│   ├── services/            # Business logic
│   ├── utils/               # Helper functions
│   └── tests/               # Test files
├── alembic/                 # Database migrations
├── requirements.txt
└── README.md

### Frontend Structure
frontend/
├── src/
│   ├── app/                 # Next.js 13+ app directory
│   ├── components/          # Reusable UI components
│   ├── lib/                 # Utility functions
│   ├── types/               # TypeScript type definitions
│   └── hooks/               # Custom React hooks
├── public/
├── package.json
└── README.md

## Implementation Guidelines

### When Writing New Features
1. Start with the simplest possible implementation
2. Write the happy path first
3. Add error handling second
4. Add tests for critical paths
5. Refactor only if complexity becomes problematic

### Database Operations
- Use transactions for multi-step operations
- Implement pagination for list endpoints
- Add database indexes for frequently queried fields
- Use connection pooling appropriately

### API Design
- Use RESTful conventions consistently
- Version your APIs (v1, v2) from the start
- Return consistent error response formats
- Include request/response examples in docstrings

### Security Considerations (Even for MVP)
- Validate all user inputs
- Use parameterized queries (SQLAlchemy handles this)
- Don't log sensitive information
- Set appropriate CORS policies
- Use HTTPS in production

## Communication Standards

### Code Comments
- Explain WHY, not WHAT the code does
- Add TODO comments for known improvements
- Document any non-obvious business logic
- Include links to relevant documentation

### Git Commit Messages
Use conventional commit format:
- `feat: add document scraping functionality`
- `fix: handle PDF parsing errors gracefully`
- `refactor: simplify analysis chain logic`
- `docs: update API documentation`

### Documentation Requirements
- README with setup instructions
- API documentation (use FastAPI's auto-generated docs)
- Environment variable documentation
- Deployment instructions

## Testing Philosophy

### What to Test (MVP Focus)
- Core business logic functions
- API endpoint happy paths
- Database model relationships
- Critical error scenarios

### What NOT to Test (For MVP)
- UI components extensively
- Third-party library functionality
- Every edge case scenario
- Performance under load

## Performance Considerations

### Backend
- Use database indexes appropriately
- Implement basic caching for expensive operations
- Use async/await for I/O operations
- Don't premature optimize - measure first

### Frontend
- Use Next.js built-in optimizations
- Lazy load components when appropriate
- Minimize bundle size with tree shaking
- Optimize images and assets

## Decision Framework

When facing any technical decision, ask:
1. **Does this solve the immediate problem?**
2. **Is this the simplest solution that works?**
3. **Can I understand this code in 6 months?**
4. **Does this follow established patterns in the codebase?**
5. **Will this make future changes easier or harder?**

Choose the solution that best balances these factors, always favoring simplicity over cleverness.

## Red Flags to Avoid

- Adding frameworks/libraries for single-use cases
- Creating abstractions before you have 3+ similar use cases
- Implementing features "because we might need them later"
- Using complex design patterns for simple problems
- Writing code that requires extensive documentation to understand

Remember: The best code is code that doesn't need to exist. The second-best code is code that's so simple it obviously works.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/midaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/midaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
