## treat

> This document outlines the core principles and guidelines for developing the Uganda OneStop Centre (OSC) Digital Tool — a hybrid monorepo with a Next.js frontend and ASP.NET Web API backend.

# CLAUDE Project Directives

This document outlines the core principles and guidelines for developing the Uganda OneStop Centre (OSC) Digital Tool — a hybrid monorepo with a Next.js frontend and ASP.NET Web API backend.

## Repository Structure
```
/
├── frontend/    ← Next.js 15 + Vercel (production app)
├── backend/     ← ASP.NET Web API + PostgreSQL
├── shared/      ← Shared types/contracts
├── CLAUDE.md
├── PROGRESS.md
└── .gitignore
```

## Core Philosophy
- **Systematic Implementation**: Build the application "Brick by Brick." Each component and feature must be fully implemented, tested, and verified before moving to the next. Do not make assumptions; validate every step.
- **Cultural Relevance**: All design, content, and functionality must be tailored to the Ugandan context. Conduct thorough research to ensure cultural sensitivity and appropriateness.
- **Hybrid Architecture**: Frontend stays on Vercel (Next.js SSR). Backend migrates incrementally to ASP.NET Web API for features requiring a real database (auth, tickets, messaging, analytics). Sanity CMS remains for content (agencies, events, investment descriptions, downloadable resources).

## Agent & Tooling
- **Chatbot LLM**: Groq API with LLaMA 3.3 70B for the investor chatbot.
- **CMS**: Sanity.io for content management.
- **Email**: Postmark for transactional email.

## Development Workflow
1.  **Frontend**: All frontend work lives in `/frontend/`. Run `npm run dev` from that directory.
2.  **Backend**: ASP.NET Web API lives in `/backend/`. Run `dotnet run` from `backend/src/OscApi/`.
3.  **Routing**: Implement and verify all internal navigation. Ensure all links, buttons, and interactive elements route to the correct pages.
4.  **Documentation**: Maintain a factual and up-to-date log of all development activities in `PROGRESS.md`.

## Code Quality & Conventions
- **Frontend**: Next.js 15 with TypeScript, Tailwind CSS, App Router.
- **Backend**: ASP.NET Web API with C#, PostgreSQL, EF Core.
- **Modules**: Use ES modules (`import`/`export`) in frontend.
- **Branding**: Strictly adhere to the Uganda flag color palette: black, yellow (gold #FFD700), red (#CE1126), and white.
- **Imagery**: Use sample images relevant to the Ugandan context.

## Testing & Verification
- **Zero Errors**: The codebase must be free of ESLint, TypeScript, and browser console errors.
- **CI/CD**: All builds must pass continuous integration checks.
- **Dependencies**: Remove any unused packages or dependencies.
- **Accessibility**: Ensure the application complies with WCAG 2.1 AA standards.

## Key Documentation
- **Progress Logs**: `PROGRESS.md`
- **Investment Migration Plan**: `INVESTMENT_MIGRATION_PLAN.md`

## Critical Reminders
- **No Assumptions**: Always verify component integrations, routes, and links.
- **Completeness**: Avoid placeholders (except for images). All features must be fully functional.
- **User-Centric**: Design and build with the end-user (investors and business owners in Uganda) in mind.

## 🔒 Security Best Practices

### Keep your secrets SECRET
- **Environment Variables**: Store API keys, tokens, and other sensitive credentials in environment variables (`.env` files), never in code.
- **Key Rotation**: Rotate all keys and tokens every few months as a security measure.

### Collect less sensitive files
- **Avoid Storing Sensitive Data**: Do not store sensitive user data directly. Use trusted third-party services like Google or Instagram for authentication and profile information where possible.
- **Payment Processing**: Never store raw credit card numbers. Use secure, compliant payment gateways like Stripe or PayPal.

### Use review
- **Sanitize Inputs**:
  - Implement CAPTCHA or reCAPTCHA to prevent automated abuse.
  - Use an SSL certificate (HTTPS) to encrypt all data in transit.
- **Monitor Actively**:
  - Use monitoring services like DataDog, Sentry.io, or your hosting platform's built-in tools to track application performance and security events.
- **Row-Level Security (RLS)**: Implement RLS in your database to ensure users can only access their own data.
- **Update Dependencies**: Regularly update all packages and dependencies to patch security vulnerabilities.
- **Rely on Trusted Providers**: Use well-vetted, reputable third-party libraries and services.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ptr234)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ptr234)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
