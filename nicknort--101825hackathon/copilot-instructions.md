## 101825hackathon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 14 application that provides a web-based chat interface to interact with Claude AI (Sonnet 3.5), designed to be deployed on Vercel. The application implements strict security controls including API key authentication, rate limiting, and hardcoded model parameters to prevent abuse.

## Development Commands

**Local Development:**
```bash
npm run dev        # Start development server on localhost:3000
npm run build      # Build production bundle
npm start          # Start production server
```

**Environment Setup:**
Create `.env.local` with:
```
ANTHROPIC_API_KEY=<your_anthropic_api_key>
ALLOWED_API_KEYS=<comma_separated_client_keys>
```

**Deployment:**
```bash
vercel             # Deploy to preview
vercel --prod      # Deploy to production
```

After deployment, set environment variables:
```bash
vercel env add ANTHROPIC_API_KEY
vercel env add ALLOWED_API_KEYS
```

## Architecture

### Application Structure

The app follows Next.js 14 App Router architecture:

- **`app/api/chat/route.ts`** - API endpoint handling all Claude AI interactions
  - Implements authentication, rate limiting, and input validation
  - Proxies requests to Anthropic API with hardcoded security parameters

- **`app/page.tsx`** - Client-side chat interface
  - React component with local state management for messages
  - Handles user input, API key collection, and message display

- **`app/layout.tsx`** - Root layout with metadata

- **`next.config.js`** - Next.js configuration with security headers
  - Sets X-Frame-Options, X-Content-Type-Options, HSTS, etc.

### Security Architecture

The application has **layered security controls** that are intentionally restrictive:

1. **Authentication Layer** (app/api/chat/route.ts:20-33)
   - All API requests require `X-API-Key` header
   - Keys validated against `ALLOWED_API_KEYS` environment variable
   - Returns 401 for invalid/missing keys

2. **Rate Limiting** (app/api/chat/route.ts:9-67)
   - In-memory rate limiting: 10 requests per minute per API key/IP
   - Automatically cleans up old entries when map exceeds 1000 entries
   - Returns 429 when limit exceeded

3. **Input Validation** (app/api/chat/route.ts:105-128)
   - Max 50 messages per conversation
   - Max 10,000 characters per message
   - Validates message array structure

4. **Parameter Hardcoding** (app/api/chat/route.ts:14-56)
   - Model: `claude-3-5-sonnet-20241022` (cannot be changed by clients)
   - Max tokens: 1000 (cannot be changed by clients)
   - System prompt: Specialized for Claude Code skill creation (cannot be changed by clients)
   - The system prompt includes comprehensive guidance on creating Claude Code skills, including skill structure, best practices, and the creation process
   - These are **intentionally immutable** to prevent cost overruns and abuse

5. **Security Headers** (next.config.js:4-32)
   - X-Frame-Options: DENY
   - Strict-Transport-Security with includeSubDomains
   - Content-Type and Referrer policies

### Data Flow

1. User enters message in `app/page.tsx`
2. Client sends POST to `/api/chat` with `X-API-Key` header and messages array
3. API route authenticates request → checks rate limit → validates input
4. API calls Anthropic SDK with hardcoded parameters
5. Response returned to client and displayed in chat interface

## Important Implementation Details

### API Route Security Pattern

The `/api/chat` endpoint uses a **defense-in-depth approach** with sequential validation:

```typescript
// Order matters - authentication before rate limiting before processing
authenticateRequest() → checkRateLimit() → validateInput() → callAnthropicAPI()
```

If modifying the API route, maintain this order and never expose detailed errors to clients (app/api/chat/route.ts:150-154).

### Client-Side State Management

The chat interface (app/page.tsx) maintains conversation state entirely in local React state - there is no backend session storage. The entire message history is sent with each request. This means:

- Conversations reset on page refresh
- No database or persistent storage
- Message history grows with each turn until page reload

### Rate Limiting Limitations

The current rate limiting implementation uses an **in-memory Map** (app/api/chat/route.ts:10), which means:

- Rate limits reset when the server/function restarts
- Does not persist across Vercel serverless function instances
- Not suitable for distributed/multi-instance deployments
- For production use at scale, consider Redis or Vercel KV

### Environment Variables

Both `ANTHROPIC_API_KEY` and `ALLOWED_API_KEYS` must be set in production:

- `ANTHROPIC_API_KEY` - Your Anthropic API key for server-side API calls
- `ALLOWED_API_KEYS` - Comma-separated list of authorized client API keys (trimmed automatically)

Missing either variable will cause runtime failures.

## Common Modification Scenarios

**To change Claude model or parameters:**
Modify constants in app/api/chat/route.ts:14-56. Note: These are intentionally hardcoded for security - client requests cannot override them.

**To customize the system prompt:**
The agent currently specializes in helping users create Claude Code skills. To change this specialization or revert to a general-purpose assistant, modify `HARDCODED_SYSTEM_PROMPT` in app/api/chat/route.ts:18-56. The current system prompt includes comprehensive knowledge about skill structure, creation process, and best practices.

**To adjust rate limits:**
Modify constants in app/api/chat/route.ts:11-12 (`RATE_LIMIT_WINDOW` and `RATE_LIMIT_MAX_REQUESTS`).

**To add more security headers:**
Edit the headers array in next.config.js:4-32.

**To change input validation limits:**
Modify validation logic in app/api/chat/route.ts:113-128.

## Tech Stack

- **Framework:** Next.js 14 (App Router with TypeScript)
- **AI SDK:** @anthropic-ai/sdk v0.27.0
- **Runtime:** Node.js 20+
- **Deployment:** Vercel (serverless functions)
- **Styling:** Inline styles (no CSS framework)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NickNort)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NickNort)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
