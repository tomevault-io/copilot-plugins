## pulse-mag

> - Never push to `main` directly - always merge from `staging`

## Pulse-Mag Project Rules

### Git & Deployment
- Never push to `main` directly - always merge from `staging`
- Branch naming: `feature/description`, `fix/description`, `hotfix/description`
- Always test on staging before promoting to main
- Commit format: `type(scope): description` (conventional commits)
- Merge flow: dev → staging → main (never skip!)

### Security
- Never commit [.env.local](cci:7://file:///i:/Programing/Projects/Pulse-Mag/apps/web/.env.local:0:0-0:0) - it's in `.gitignore` for a reason
- Never use `NEXT_PUBLIC_` prefix for API keys, auth tokens, or secrets
- Only `NEXT_PUBLIC_` for site URLs and public configuration
- Never expose Wix dashboard URL publicly - use pulseliterary.com only

### Content Management
- Blog posts: Written in Wix, displayed via RSS
- Magazine issues: Uploaded to Sanity CMS
- Static pages: Built in Next.js (About, Submit, Join, etc.)
- Authors: Managed in Sanity CMS
- Never edit blog posts in code - always use Wix dashboard

### Environments
- Production (`main`): pulseliterary.com - public live site
- Staging (`staging`): Preview URL - testing before release
- Dev (`dev`): Preview URL - active development
- Always verify environment variables are set before deploying

### Code Quality
- TypeScript everywhere - no `any` types
- Use existing components - maintain design system
- Follow Tailwind patterns - use project color tokens
- No `console.log` in production - use proper error handling
- Test Wix RSS connection before marking blog work complete

### Domain & URLs
- Public facing: Always share `pulseliterary.com`
- Private editing: Wix dashboard for blog posts only
- No Wix redirects - Premium Core doesn't support 301 redirects
- Never link to pulse24.wixsite.com in public communications

### API Usage
- RSS first - try RSS before Wix API (more reliable)
- Revalidate every 5 min - don't hammer Wix servers
- Graceful fallbacks - if Wix fails, show friendly error
- Cache responses - use Next.js revalidation

### Emergency Procedures
- If production breaks: Hotfix branch from main, fix, merge back
- If Wix goes down: Site still works, just no new blog posts
- If build fails: Check env variables first, then check Wix RSS URL

### Performance
- Use `next/image` for all images (never `<img>`)
- Set appropriate `revalidate` times (60s blog, longer for static)
- Lazy load below-the-fold content
- Test Lighthouse score before major releases

### Pre-Deploy Checklist
- Test on staging URL before merging to main
- Verify mobile layout works
- Check that Wix RSS connection still fetches posts
- Confirm no `console.log` statements in production code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/McCal-Codes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
