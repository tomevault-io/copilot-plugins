## sogni-demo-superapp

> This is a teaching demo showcasing the Sogni AI render network platform through a clean, production-ready full-stack application. The app generates tattoo concept art using AI rendering with real-time progress streaming.

# Sogni Demo Superapp - Cursor AI Rules
# A Node.js Express + React + TypeScript + Sogni SDK demo application

## Project Overview
This is a teaching demo showcasing the Sogni AI render network platform through a clean, production-ready full-stack application. The app generates tattoo concept art using AI rendering with real-time progress streaming.

## Stack & Architecture
- **Backend**: Node.js + Express (ES modules), Sogni Client SDK, Server-Sent Events
- **Frontend**: React 18 + TypeScript + Vite, responsive design with accessibility
- **Package Manager**: pnpm workspace (monorepo structure)
- **Communication**: REST API + SSE for real-time streaming
- **Styling**: Inline CSS with design system variables, mobile-first responsive

## Core Principles
1. **Keep it simple and teachable** - This is a demo/teaching app, prioritize clarity over complexity
2. **Maintain clean separation** - Backend handles Sogni SDK, frontend handles UX
3. **Fire-and-stream pattern** - POST to start, SSE to stream progress/results
4. **Production-ready patterns** - Even though it's a demo, use production best practices

## File Structure Rules
```
sogni-demo-superapp/
├── server/           # Express API (Sogni SDK integration)
│   ├── index.js     # Main server file (ES modules)
│   ├── package.json # Server dependencies
│   └── .env         # Server environment (gitignored)
├── web/             # React frontend
│   ├── src/
│   │   ├── main.tsx    # App entry point
│   │   ├── App.tsx     # Main component (keep monolithic for teaching)
│   │   └── components/ # Create this when App.tsx > 500 lines
│   ├── package.json    # Frontend dependencies
│   └── .env           # Frontend environment (gitignored)
└── package.json      # Root workspace config
```

## Coding Standards

### General
- Use 2-space soft tabs consistently across all files
- Prefer explicit imports over default exports for better IDE support
- Add JSDoc comments for complex functions and API endpoints
- Keep functions focused and under 50 lines when possible

### TypeScript/JavaScript
- Use TypeScript strict mode in frontend
- Use ES modules syntax consistently (`import`/`export`)
- Prefer `const` over `let`, avoid `var`
- Use meaningful variable names that explain intent
- Type React components properly with interfaces for props

### React Patterns
- Keep App.tsx as main component until it exceeds 500 lines
- Use functional components with hooks
- Implement proper error boundaries for production readiness
- Use React.memo() for expensive components
- Maintain accessibility with proper ARIA attributes and semantic HTML

### Backend Patterns
- Keep server/index.js as single file until it exceeds 800 lines
- Use proper error handling with try/catch blocks
- Implement CORS properly for security
- Use environment variables for all configuration
- Follow REST conventions for API endpoints

### State Management
- Use React useState for simple state
- Consider useReducer for complex state transitions
- Keep state close to where it's used
- Use SSE for real-time updates, not polling

## Context Window Management

### File Size Guidelines
- **Critical**: Split files when they exceed these limits:
  - React components: 500 lines → create components/ subdirectory
  - Server files: 800 lines → create routes/, middleware/, utils/ subdirectories
  - Any file: 1000 lines → mandatory split

### Code Organization
- Group related functionality together
- Extract reusable utilities to separate files
- Use barrel exports (index.js) for clean imports
- Keep configuration at the top of files

### When Splitting Components
```typescript
// Before (App.tsx > 500 lines)
export default function App() { /* ... */ }

// After
// src/components/TattooForm.tsx
// src/components/IdeaGrid.tsx
// src/components/ProgressStream.tsx
// src/App.tsx (orchestrates components)
```

### When Splitting Server
```javascript
// Before (server/index.js > 800 lines)

// After
// server/routes/generate.js
// server/routes/progress.js
// server/middleware/cors.js
// server/utils/sogni.js
// server/index.js (orchestrates modules)
```

## API Design Patterns

### Endpoints
- `POST /api/generate` - Start render with Sogni SDK
- `GET /api/progress/:projectId` - SSE stream for real-time updates
- `GET /api/cancel/:projectId` - Cancel running project
- `GET /api/result/:projectId/:jobId` - Proxy result images
- `GET /api/health` - Health check

### Error Handling
- Always return consistent error format: `{ error: string }`
- Use appropriate HTTP status codes
- Handle Sogni SDK errors gracefully
- Log errors server-side, never expose internal details to client

### SSE Best Practices
- Send heartbeat every 20 seconds to prevent proxy timeouts
- Clean up connections properly on client disconnect
- Handle connection drops gracefully with reconnection logic
- Use structured event format: `{ type: string, ...data }`

## Sogni SDK Integration

### Authentication
- Store credentials in server environment only
- Never expose API keys to frontend
- Use lazy client initialization to avoid startup issues
- Handle "Project not found" errors gracefully (common SDK noise)

### Project Management
- Use fire-and-stream pattern: create project, then stream events
- Store active projects in memory (Map) for result URL access
- Clean up project references when completed/failed
- Use Spark tokens for billing (tokenType: 'spark')

### Event Handling
- Forward all Sogni events to frontend via SSE
- Normalize progress values (0-1 for SDK compatibility)
- Provide both modern and legacy event formats for compatibility
- Handle preview and final result URLs properly

## Performance Guidelines

### Frontend
- Use React.memo() for components that re-render frequently
- Implement proper loading states and skeleton UIs
- Lazy load images with loading="lazy"
- Use proper image sizing and aspect ratios

### Backend
- Reuse single Sogni client instance
- Implement proper connection pooling
- Use streaming for large responses
- Cache static assets with appropriate headers

### Bundle Optimization
- Keep dependencies minimal and purposeful
- Use Vite's built-in optimizations
- Implement proper code splitting when app grows
- Monitor bundle size with `pnpm build` analysis

## Security Considerations

### Environment Variables
```bash
# Server (.env)
SOGNI_USERNAME=your_username
SOGNI_PASSWORD=your_password
SOGNI_APP_ID=sogni-tattoo-ideas
SOGNI_ENV=production
PORT=3001
CLIENT_ORIGIN=http://localhost:5173

# Web (.env)
VITE_API_BASE_URL=https://your-api.example.com  # Optional
```

### CORS Configuration
- Explicitly whitelist allowed origins
- Never use wildcard (*) in production
- Handle preflight requests properly
- Validate origin header server-side

## Development Workflow

### Adding Features
1. Start with the simplest possible implementation
2. Add proper TypeScript types
3. Implement error handling
4. Add loading states and user feedback
5. Test on mobile devices
6. Document any new environment variables

### Debugging
- Use browser DevTools for frontend issues
- Use Node.js debugger for backend issues
- Monitor SSE connections in Network tab
- Check server logs for Sogni SDK errors

### Testing Approach
- Manual testing for UI/UX flows
- Test SSE connections with multiple clients
- Verify CORS with different origins
- Test error scenarios (network failures, invalid inputs)

## Common Pitfalls to Avoid

### Context Window Issues
- ❌ Don't read entire large files when debugging
- ✅ Use targeted searches and specific line ranges
- ❌ Don't create overly complex single files
- ✅ Split files proactively at size limits

### React Anti-patterns
- ❌ Don't mutate state directly
- ✅ Always use setState functions
- ❌ Don't forget dependency arrays in useEffect
- ✅ Include all dependencies or use useCallback

### Backend Issues
- ❌ Don't block the event loop with synchronous operations
- ✅ Use async/await for all I/O operations
- ❌ Don't forget to handle promise rejections
- ✅ Implement proper error boundaries

### Sogni SDK Issues
- ❌ Don't create multiple client instances
- ✅ Reuse single client with lazy initialization
- ❌ Don't ignore "Project not found" noise
- ✅ Filter known SDK noise in error handling

## Scaling Considerations

### When to Refactor
- App.tsx > 500 lines: Extract components
- server/index.js > 800 lines: Extract routes/middleware
- More than 3 similar components: Create shared component
- Repeated logic: Extract to utility functions

### Database Integration (Future)
- Add user accounts and project persistence
- Store render history and favorites
- Implement rate limiting per user
- Add project sharing capabilities

### Deployment Optimization
- Use environment-specific builds
- Implement proper logging and monitoring
- Add health checks and graceful shutdowns
- Use CDN for static assets and images

## IDE Integration

### Recommended Extensions
- ES6 String HTML (for styled-components if added)
- Auto Rename Tag
- Bracket Pair Colorizer
- GitLens
- Thunder Client (for API testing)

### Settings
- Enable format on save
- Use 2-space indentation
- Enable TypeScript strict mode
- Show whitespace characters

## Error Messages
When encountering errors, provide context about:
1. Which part of the stack (frontend/backend/Sogni SDK)
2. Current operation being performed
3. Relevant environment variables or configuration
4. Whether it's a known Sogni SDK noise issue

## Final Notes
- This is a teaching demo - prioritize code clarity over optimization
- Keep the core fire-and-stream pattern intact
- Document any deviations from these patterns
- Maintain the clean separation between frontend and backend concerns
- Always test the full flow: form → API → Sogni → SSE → UI updates

---
> Source: [blkluv/sogni-demo-superapp](https://github.com/blkluv/sogni-demo-superapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
