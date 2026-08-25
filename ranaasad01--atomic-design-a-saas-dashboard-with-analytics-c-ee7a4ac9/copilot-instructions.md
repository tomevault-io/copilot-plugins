## atomic-design-a-saas-dashboard-with-analytics-c-ee7a4ac9

> Project conventions for AI agents and humans editing this codebase.

# AGENTS.md

Project conventions for AI agents and humans editing this codebase.

## Original request
Design a SaaS dashboard with analytics charts

## Goal
Build a glass-futuristic SaaS analytics dashboard with KPI cards, interactive Recharts visualizations, sidebar navigation, and four fully-functional pages: Dashboard, Analytics, Users, and Settings.

## Project type
dashboard

## Design system — match this exactly
- Color tokens: `--background: #0F172A`, `--foreground: #F8FAFC`, `--card: #1E293B`, `--border: #334155`, `--muted-foreground: #94A3B8`, `--primary: #6366F1`, `--accent: #22D3EE`, `--brand-primary: #1E40AF`, `--on-primary: #FFFFFF`, `--brand-secondary: #3B82F6`, `--brand-accent: #D97706`, `--brand-background: #F8FAFC`

## Existing components — reuse these, don't create near-duplicates
- Footer (components/Footer.tsx)
- LanguageToggle (components/LanguageToggle.tsx)
- LocaleProvider (components/LocaleProvider.tsx)
- Navbar (components/Navbar.tsx)

## Existing i18n namespaces
Every translation key must be namespaced (`hero.title`, never a bare `title`) so two components never collide on the same catalog slot. Reuse one of these, or pick a new, distinct name:
`analytics`, `cta`, `dashboardOverview`, `features`, `hero`, `logos`, `metrics`, `nav`, `pricing`, `settings`, `testimonials`, `users`

When editing or adding pages: preserve the design system above, reuse existing components and the shared nav data file, and keep the established structure and tone.

---
> Source: [ranaasad01/atomic-design-a-saas-dashboard-with-analytics-c-ee7a4ac9](https://github.com/ranaasad01/atomic-design-a-saas-dashboard-with-analytics-c-ee7a4ac9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
