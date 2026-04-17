## newsletter-rss-reader

> Cursor rules for React + Next 15 Edge architecture on Cloudflare


# Architecture Rules
- React + Next.js 15 deployed on Cloudflare Pages via @cloudflare/next-on-pages
- Edge runtime for all API routes and server components
- Convex for real-time data (engagements, metadata, auth)
- PlanetScale for RSS entries via Hyperdrive
- Cloudflare ecosystem: Queues, Workers, Images, Hyperdrive, Axiom logs

# Core Principles
- Never over-engineer solutions - prefer simple, direct implementations
- No testing scripts unless explicitly requested
- No performance monitoring scripts unless explicitly requested
- Always apply logical decision-making to code choices
- Scale-first mindset for 100k concurrent users

# React Best Practices
- Refer to Docs when needed
- Use React 19 features
- Implement proper error boundaries for resilient UX
- Optimize with React.memo, useMemo, useCallback only when needed
- Only use useEffect where deemed appropriate by React best practices

# Database Strategy
- Convex for real-time, collaborative features and auth
- PlanetScale via Hyperdrive for high-volume RSS data
- Use proper indexing and query optimization
- Use Hyperdrive for database connection pooling

# Code Style
- TypeScript strict mode enabled
- Use centralized types from types files
- Avoid 'any' type - use proper typing
- Prefer async/await over promises
- Use proper error handling patterns

# When Unsure
- Consult official documentation first
- Check Cloudflare compatibility before implementing
- Verify edge runtime compatibility
- Test on edge before production deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicalevras) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
