## dailyreflections

> Daily Reflections is a recovery-focused web app showing daily AA readings with threaded comments.

# Daily Reflections App - Project Intelligence

## Project Context
Daily Reflections is a recovery-focused web app showing daily AA readings with threaded comments.

## Key Patterns

### MongoDB Connection
ALWAYS use `@/lib/mongodb` import for database access.
Connection is pooled and reused across requests.

### Date Keys
Use "MM-DD" format consistently (e.g., "01-08").
Store in `dateKey` field across collections.

### Component Naming
- ReflectionCard: Main reflection display
- CommentList: Container for comments
- CommentItem: Individual comment with recursion
- CommentForm: Comment submission

### Material UI
Prefer Material UI components over custom styling.
Use theme provider for consistent colors.

### API Routes
Put all server logic in `/app/api` routes.
Use proper HTTP methods (GET, POST, PUT, DELETE).

### Error Handling
Always wrap DB calls in try/catch.
Return consistent error responses.
Show user-friendly error messages.

### Security
Sanitize all HTML with DOMPurify.
Validate all user input.
Never expose sensitive data.

## User Preferences
- JavaScript over TypeScript
- Material UI over Tailwind
- MongoDB Atlas (never local)
- Clean, modern UI
- Full-screen details display

## Code Quality
- Clear, readable code
- Consistent naming
- Proper indentation
- Comment complex logic
- Modular structure

## Testing Approach
- Manual component testing
- API endpoint validation
- Database query verification
- Responsive design checks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrlynn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
