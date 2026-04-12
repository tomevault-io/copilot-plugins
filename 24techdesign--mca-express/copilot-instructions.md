## mca-express

> - Next.js 14 (App Router)

# MCA Express Project Rules

## Technical Stack
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS

## Content Locations
- Contact info: `app/page.tsx` (lines 488-522)
- Services: `app/page.tsx` (lines 38-63)

## Verification Workflow (Cloud Optimized)
When running as an automated agent in GitHub Actions:

1. **Understand** - Read the linked issue to identify the requested change
2. **Execute** - Modify only the necessary files
3. **Verify** - Run lint and build:
   - Execute: `pnpm run lint` (must pass)
   - Execute: `pnpm run build` (must pass)
4. **Skip screenshots** - Do NOT attempt to take screenshots. The Playwright setup is complex and causes failures.
5. **Create PR** - Use `gh pr create` with proper title and body
6. **Safety** - If `lint` or `build` fails:
   - Do NOT open a PR
   - Leave a comment on the original issue explaining the error

Do NOT create a PR if lint or build fails.

## Performance Notes
- Dependencies are pre-installed by the workflow (skip `pnpm install`)
- Next.js uses turbo mode automatically for faster builds
- Always keep the build step - it catches errors before Netlify deployment
- Screenshots are disabled to avoid complexity and failures

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/24techdesign)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/24techdesign)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
