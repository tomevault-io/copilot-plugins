## clawsync

> ClawSync is an open source AI agent platform built with React 18 + TypeScript + Vite (frontend) and Convex (backend). The agent uses @convex-dev/agent for thread management, tool calling, and real-time streaming. Multi-model support via Vercel AI SDK.

# CLAUDE.md

## Project
ClawSync is an open source AI agent platform built with React 18 + TypeScript + Vite (frontend) and Convex (backend). The agent uses @convex-dev/agent for thread management, tool calling, and real-time streaming. Multi-model support via Vercel AI SDK.

## Commands
```bash
npm install          # Install dependencies
npx convex dev       # Start Convex backend (runs alongside frontend)
npm run dev          # Start Vite dev server
npm run build        # Production build
npx convex deploy    # Deploy Convex functions to production
npm test             # Run tests
npm run lint         # ESLint
npm run typecheck    # TypeScript type check
```

## Key files
- `convex/schema.ts` - Database schema. All tables, all indexes.
- `convex/agent/clawsync.ts` - Agent definition. Tools loaded dynamically, not hardcoded.
- `convex/agent/security.ts` - Security checker. Runs on EVERY tool invocation. Do not bypass.
- `convex/agent/toolLoader.ts` - Assembles tools from skillRegistry + MCP servers at runtime.
- `convex/agent/modelRouter.ts` - Resolves provider + model from agentConfig.
- `convex/auth.config.ts` - WorkOS AuthKit JWT config (placeholder, not yet active).
- `convex/xTwitter.ts` - X/Twitter API v2 integration (read, reply, post tweets).
- `convex/crons.ts` - Background jobs (summary tables, health checks, log cleanup).
- `content/soul.md` - Default agent identity. Fork owners customize this.
- `src/styles/tokens.css` - Design system CSS custom properties (Geist fonts).
- `features.html` - Standalone features page for browser viewing.

## Hosting rules (non-negotiable)
- This project uses **Convex Self Static Hosting ONLY**
- Do NOT add Vercel, Netlify, or other external hosting configs
- Component: `@convex-dev/self-static-hosting` (installed from GitHub)
- See: https://github.com/get-convex/self-static-hosting

### Deployment options (in order of preference):
1. **Convex Storage** - Files in Convex, served via HTTP actions (simplest)
2. **Convex + Cloudflare CDN** - Files in Convex, cached at edge (custom domains)
3. **Cloudflare Pages** - Files at Cloudflare edge (best performance)

### Key files:
- `convex/staticHosting.ts` - Upload API for self-static-hosting
- `convex/convex.config.ts` - Component registration
- `convex/http.ts` - Static routes via `registerStaticRoutes()`

### Deploy commands:
```bash
npm run deploy         # Build + deploy frontend to Convex
npm run deploy:static  # Upload static files only
```

## Authentication
- **Current**: Password-based auth for SyncBoard (`SYNCBOARD_PASSWORD_HASH` env var)
- **Future**: WorkOS AuthKit for enterprise SSO (not yet configured)
- Auth config placeholder: `convex/auth.config.ts`
- Frontend prep: `src/main.tsx` has comments for AuthKitProvider setup
- See: https://docs.convex.dev/auth/authkit/

## X (Twitter) Integration
- Config stored in `xConfig` table, managed via SyncBoard > X
- Tweets cached in `xTweets` table for landing page display
- Required env vars: `X_BEARER_TOKEN`, `X_API_KEY`, `X_API_SECRET`, `X_ACCESS_TOKEN`, `X_ACCESS_TOKEN_SECRET`
- Rate limiting: configurable per hour in SyncBoard
- See: https://developer.x.com/en/docs/x-api

## Model Providers
Supported providers (set API key in Convex env vars):
- **Anthropic**: `ANTHROPIC_API_KEY` - Claude models
- **OpenAI**: `OPENAI_API_KEY` - GPT models
- **xAI**: `XAI_API_KEY` - Grok models
- **OpenRouter**: `OPENROUTER_API_KEY` - 300+ models

## Convex components
- `convex/convex.config.ts` is the single registration point for all components.
- Core: @convex-dev/agent, @convex-dev/rate-limiter, @convex-dev/self-static-hosting.
- Access via `components` from `"./_generated/api"` (e.g., `components.rateLimiter`).
- Components are sandboxed. They can't access your tables or env vars unless passed in.
- To add a new component: `npm install`, add `app.use()` to convex.config.ts, run `npx convex dev`.
- Never reach into a component's internal files. Use the public API on the `components` object.
- After adding/removing a component, always run `npx convex dev` to regenerate types.

## Convex patterns to follow
- Always validate function arguments with Convex validators (`v.object({})`)
- Always use indexes in queries. Define them in schema.ts.
- Never use `.collect()` on tables that grow. Use `.take(n)` or paginate.
- Dashboard reads from summary tables (skillInvocationSummary), not from hot log tables.
- Store large outputs (>10KB) in Convex file storage, put storage ID in the log.
- Internal functions: `internalMutation`, `internalAction`, `internalQuery`.
- Cron jobs go in `convex/crons.ts` and run isolated from real-time traffic.
- HTTP endpoints defined in `convex/http.ts`.

## Security rules (non-negotiable)
- Do NOT modify `convex/agent/security.ts` without explicit user approval and explanation.
- Do NOT store API keys, tokens, or secrets in source code, schema config fields, or git.
- Do NOT return decrypted secret values from any Convex function.
- Do NOT bypass the security checker for any tool invocation.
- All new skills must default to `approved: false` and `status: "pending"`.
- Webhook handlers MUST verify platform-specific signatures before processing.
- Domain allowlist MUST be checked on every webhook skill invocation, not just at creation.
- No HTTP redirects followed in webhook caller (SSRF prevention).
- Secrets stored in `skillSecrets` or `channelSecrets` tables, encrypted.

## Design system
Use CSS custom properties from `src/styles/tokens.css`. Never hardcode color values.
- `--interactive: #ea5b26` for buttons, active states, links
- `--text-primary: #232323` for body text
- `--bg-primary: #f3f3f3` for page backgrounds
- `--font-sans: 'Geist'` for UI text (loaded via CDN in index.html)
- `--font-mono: 'Geist Mono'` for code blocks

## What NOT to do
- Don't use `.collect()` without `.take(n)` on any table
- Don't hardcode model IDs in agent definition (they're in agentConfig table)
- Don't put tools directly in the Agent constructor (they're loaded dynamically)
- Don't store secrets in the `config` field of skillRegistry
- Don't skip input validation on Convex functions
- Don't create queries without indexes
- Don't modify security.ts without asking first
- Don't add Vercel, Netlify, or external hosting configs (use Convex Self Static Hosting only)

---
> Source: [waynesutton/clawsync](https://github.com/waynesutton/clawsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
