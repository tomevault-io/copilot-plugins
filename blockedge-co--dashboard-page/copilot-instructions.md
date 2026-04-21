## dashboard-page

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- `npm run dev`: Start development server on port 3000
- `npm run build`: Build production application
- `npm run start`: Start production server
- `npm run lint`: Run ESLint checks
- `npm run type-check`: Run TypeScript type checking
- `npm run test`: Run Jest unit tests
- `npm run test:e2e`: Run Playwright end-to-end tests
- `npm run test:mobile`: Run mobile-specific tests
- `npm run production-ready`: Full production validation (type-check, lint, build)

## Project Architecture

### High-Level Structure

This is a Next.js 14 App Router application for carbon credit trading analytics, built with TypeScript and Tailwind CSS. The architecture follows a mobile-first, enterprise-grade design with real-time blockchain data integration.

### Key Architectural Patterns

**App Router Structure**: Uses Next.js App Router with nested layouts and colocated components. Main routes include `/dashboard`, `/projects`, `/analytics`, `/grafana`, and `/reports`.

**Component Organization**: 
- `components/ui/` - shadcn/ui components (40+ reusable components)
- `components/panels/` - Dashboard-specific panels
- `components/retirement-panels/` - Retirement workflow components
- Root-level dashboard components for different views

**Data Layer**: 
- `lib/co2e-api.ts` - Primary blockchain API integration
- `lib/blockchain-utils.ts` - Blockchain data utilities
- `lib/retirement-service/` - Retirement data services
- `lib/tokenization-service/` - Token management
- Real-time data from CO2e Chain API and BlockEdge project data

**Testing Strategy**: 
- Jest + React Testing Library for unit tests (70% coverage threshold)
- Playwright for E2E testing across multiple browsers
- Custom mobile verification suite with screenshot comparisons

### Mobile-First Design

The application is built mobile-first with:
- Responsive breakpoints and touch optimization
- Mobile-specific test suite
- Performance optimizations for mobile devices
- Accessibility features (ARIA, keyboard navigation)

## Development Guidelines

### Code Style
- TypeScript strict mode enabled
- Use shadcn/ui components for consistency
- Follow existing component patterns for dashboard panels
- Implement proper error boundaries for fault tolerance
- Use React Hook Form with Zod validation for forms

### Data Integration
- Primary API: `https://exp.co2e.cc/api/v2` for blockchain data
- Project data: `https://asset.blockedge.co/blockedge-co2e-project.json`
- Implement proper caching strategies for performance
- Use type-safe API integration patterns

### Performance Considerations
- Components use lazy loading with code splitting
- Images optimized through Next.js image optimization
- Animations use Framer Motion for performance
- Bundle splitting by route for optimal loading

### Testing Requirements
- Unit tests required for new components and utilities
- E2E tests for critical user flows
- Mobile testing for responsive components
- Maintain 70% test coverage across all metrics

## Key Components and Services

### Dashboard Components
- Various dashboard variants (`*-dashboard.tsx` files)
- Panel components for different data visualizations
- Retirement-specific panels for certificate workflows

### Core Services
- `co2e-api.ts` - Primary blockchain API integration
- `blockchain-utils.ts` - Utility functions for blockchain operations
- Retirement and tokenization services for specialized workflows

### UI Components
- Complete shadcn/ui component library
- Custom themed components with dark/light mode support
- Recharts integration for data visualizations

## Security and Production

### Security Features
- CSP headers configured in Next.js config
- X-Frame-Options: DENY for iframe protection
- Input validation with Zod schemas
- No secrets exposed in client-side code

### Production Optimization
- Standalone output mode for containerization
- SWC compiler for optimal performance
- Progressive Web App capabilities
- SEO optimization with proper metadata

## Common Development Tasks

### Adding New Dashboard Panels
1. Create component in `components/panels/`
2. Follow existing panel patterns for consistency
3. Add proper TypeScript types
4. Include mobile-responsive design
5. Add unit tests with React Testing Library

### Integrating New Data Sources
1. Add API integration to `lib/` directory
2. Create proper TypeScript interfaces
3. Implement caching strategy
4. Add error handling and loading states
5. Write integration tests

### Extending UI Components
1. Use shadcn/ui patterns for consistency
2. Follow existing theme configuration
3. Ensure accessibility compliance
4. Add responsive behavior
5. Include visual regression tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blockedge-co) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
