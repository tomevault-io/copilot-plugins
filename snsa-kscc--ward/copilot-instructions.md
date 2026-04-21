## ward

> - **ALWAYS** use `pnpm` - never npm or yarn


# WARD Development Rules

## Package Manager

- **ALWAYS** use `pnpm` - never npm or yarn
- Install: `pnpm install`
- Dev server: `pnpm dev`
- Build: `pnpm build`
- Format: `pnpm format`

## Clampwind Usage Rules

### Text Sizes

- Use CSS variables format: `text-[clamp(var(--text-base),var(--text-lg))]`
- Never use --spacing- variables for text sizing
- Reference existing patterns in Hero.astro component

### Spacing

- Use unitless numbers format: `my-[clamp(40,72)]`, `px-[clamp(6,10)]`, `py-[clamp(2,3)]`
- Do not use --spacing- variables - they are NOT used in clampwind
- Follow the README documentation patterns

## Component Development

- Use TypeScript for ALL new code
- Implement CVA (class-variance-authority) for component variants
- Use Radix UI for accessible primitives
- Apply tailwind-merge for conditional className merging
- Follow mobile-first responsive design

## Animation Rules

- Use GSAP for complex animations
- Implement Lenis for smooth scrolling
- Use `client:load`, `client:idle`, or `client:visible` directives for Astro components
- Add reduced motion support for accessibility
- Ensure performance with `will-change` and `transform3d`

## Database Operations

- Use Drizzle ORM for type-safe operations
- Always run `pnpm generate` after schema changes
- Use environment-specific configs (dev vs prod)
- Test connections with `pnpm studio:dev`

## Code Style

- Run `pnpm format` before commits
- Use `pnpm check` for Astro type validation
- Follow existing file structure and naming conventions
- Test clampwind values on actual devices, not just dev tools

## Client-Side Integration

- Wrap GSAP/Lenis initialization in client directives
- Use `client:visible` for below-the-fold animations
- Ensure all interactive components have proper client hydration

## Environment Setup

- Copy `.env.example` to `.env` for local development
- Required vars: `DATABASE_URL`, `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `ASTRO_PORT`, `NODE_ENV`
- Use production configs for deployment builds

## Troubleshooting Checklist

- GSAP not working? → Check client directives
- Clampwind static? → Verify PostCSS config includes clampwind()
- Tailwind classes missing? → Ensure @theme directive in global CSS
- Database connection failed? → Test with `pnpm studio:dev`
- Smooth scrolling broken? → Ensure Lenis initialized client-side only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/snsa-kscc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
