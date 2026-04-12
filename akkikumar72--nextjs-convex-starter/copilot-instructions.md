## nextjs-convex-starter

> Clerk Authentication Guidelines


# Clerk Authentication Guidelines

## Integration

- Use Clerk for all authentication needs
- Integrate with Convex for user data synchronization
- Implement proper middleware for route protection
- Use Clerk's React components for auth UI

## User Management

- Store user data in Convex `users` table
- Sync Clerk user data with Convex on authentication
- Use Clerk's user ID as the primary identifier
- Implement proper user profile management

## Route Protection

- Protect dashboard routes with authentication middleware
- Use Clerk's `auth()` function for server-side protection
- Implement proper redirects for unauthenticated users
- Handle authentication state in components

## Convex Integration

- Use `convex/auth.config.ts` for Clerk integration
- Implement proper user context in Convex functions
- Use `ctx.auth.userId` for user-specific operations
- Handle authentication errors gracefully

## Security Best Practices

- Never expose sensitive user data in client code
- Validate user permissions on the server side
- Use proper CORS configuration
- Implement rate limiting for sensitive operations
- Handle authentication tokens securely

## User Experience

- Provide clear authentication flows
- Implement proper loading states during auth
- Handle authentication errors gracefully
- Provide feedback for successful operations
- Implement proper logout functionality

## Development

- Use Clerk's development environment for testing
- Test authentication flows thoroughly
- Mock authentication in tests when appropriate
- Use proper environment variables for Clerk configuration

## Error Handling

- Handle authentication failures gracefully
- Provide meaningful error messages
- Implement proper fallback behavior
- Log authentication issues for debugging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akkikumar72)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akkikumar72)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
