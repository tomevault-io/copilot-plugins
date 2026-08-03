## libra-credito-landing-page-22

> This is a React/TypeScript landing page for Libra Crédito, a loan simulation service. The project uses Vite, Tailwind CSS, and integrates with Supabase for complete backend functionality including tracking, blog CMS, and partner management.

# Claude Instructions for Libra Crédito Landing Page

## Project Overview
This is a React/TypeScript landing page for Libra Crédito, a loan simulation service. The project uses Vite, Tailwind CSS, and integrates with Supabase for complete backend functionality including tracking, blog CMS, and partner management.

## Key Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run typecheck` - Run TypeScript checks

## Project Structure
- `/src/components/` - React components
- `/src/pages/` - Page components (Index, Simulacao, AdminDashboard, Blog, Parceiros)
- `/src/services/` - API services (simulation, blog, partners, Supabase)
- `/src/utils/` - Utility functions and validations
- `/src/hooks/` - Custom React hooks
- `/public/` - Static assets and images

## Important Documentation Files
- `README.md` - Complete project documentation and setup guide
- `QUICK_START.md` - 5-minute setup guide
- `supabase-setup-complete.sql` - Complete Supabase database setup
- `supabase-fix-columns.sql` - Database column fixes (if needed)

## Key Features
- **Loan Simulation**: Intelligent credit simulation with external API + local fallback
- **360° User Tracking**: Complete user journey tracking with UTMs, referrer, device info
- **Admin Dashboard**: Real-time simulation management with filters and CSV export
- **Blog CMS**: Complete blog system with Supabase backend and image upload
- **Partner System**: Partner registration form with CRM integration ready
- **LGPD Compliant**: Privacy-compliant data handling and retention policies

## Database Tables (Supabase)
- `simulacoes` - Credit simulations with user data
- `user_journey` - Complete user tracking and analytics
- `blog_posts` - Blog posts with CMS functionality
- `blog_categories` - Blog categories management
- `parceiros` - Partner registrations

## Important URLs
- `/` - Homepage
- `/simulacao` - Credit simulation form
- `/admin` - Admin dashboard
- `/blog` - Blog listing
- `/parceiros` - Partner registration
- `/test-supabase` - System diagnostics

## Testing
Always run lint and typecheck after making changes:
```bash
npm run lint
npm run typecheck
```

## Setup Requirements
1. Install dependencies: `npm install`
2. Setup Supabase: Execute `supabase-setup-complete.sql` in Supabase SQL Editor
3. Configure Storage: Create 'blog-images' bucket in Supabase Storage
4. Test system: Visit `/test-supabase` after `npm run dev`

## Deployment
The project is configured for Vercel deployment with custom headers and redirects. All environment variables should be configured in production.

---
> Source: [cwolfbr/libra-credito-landing-page-22](https://github.com/cwolfbr/libra-credito-landing-page-22) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
