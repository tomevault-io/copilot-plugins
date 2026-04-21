## property-hub-frontend

> ZeroVacancy is a two-sided, real estate-focused creator marketplace built to streamline content production for multifamily and commercial property teams. The platform connects property managers and developers with verified local content creators — including photographers, drone operators, video editors, and 3D tour specialists — and manages the entire lifecycle from project posting through to payment and final content delivery.

# ZeroVacancy Frontend

## Project Overview
ZeroVacancy is a two-sided, real estate-focused creator marketplace built to streamline content production for multifamily and commercial property teams. The platform connects property managers and developers with verified local content creators — including photographers, drone operators, video editors, and 3D tour specialists — and manages the entire lifecycle from project posting through to payment and final content delivery.

## 🔑 Core Features
- **Creator Discovery**: Visual-first search experience with filters by service, location, price, and availability.
- **Project Posting**: Managers can post project briefs and receive applications from multiple creators.
- **Direct Booking**: Property teams can book creators directly with a request form.
- **Dashboards**:
  - **Creator Dashboard**: Track bookings, manage portfolio, and set availability.
  - **Manager Dashboard**: Post projects, view applicants, manage payments and deliverables.
  - **Admin Dashboard**: Internal moderation tools for managing disputes, verifying creators, and monitoring platform activity.
- **Notifications**: Visual feedback system (toast alerts + nav bell) for all user actions.
- **Payments (UI-Only)**: Visual scaffolding for a Stripe Connect-style payment system with manager-side tracking and creator-side earnings reports.

## ⚙️ Tech Stack
- **Framework**: React + Vite
- **Styling**: Tailwind CSS + shadcn/ui + custom design system
- **Routing**: React Router DOM with explicit route declarations
- **Component Architecture**: Modular, token-driven system with clean separation by UI, cards, forms, layout, and dashboards
- **State Management**: Minimal local state for now (context/hooks), ready for scaling to global store
- **Language**: TypeScript
- **Form Handling**: React Hook Form with Zod validation
- **Data Fetching**: React Query (TanStack Query)
- **Date Handling**: date-fns
- **Charts**: Recharts

## 🧱 Codebase Goals
- Scalable, maintainable structure using a feature-based folder system
- Fully tokenized design system (typography, spacing, radius, colors)
- All components reusable and centralized (buttons, tags, cards, forms)
- Clean import paths and dead-simple onboarding for new devs

## 🚀 Status
MVP functionality is built and structured, including:
- Creator profiles and portfolios
- Project creation and bidding
- Booking requests and form flows
- Application review workflows
- Payment and earnings visualization
- Admin ops tooling and dashboard structure
Backend integration (auth, real-time, payments) will be introduced in Phase 2.

## Project Structure
- `/src`: Main source code
  - `/app`: Folder structure mimicking Next.js-style routing (logical organization only)
  - `/components`: Reusable UI components
  - `/hooks`: Custom React hooks
  - `/lib`: Utility functions and helpers
  - `/pages`: Page components for different routes
  - `/styles`: Global styles and Tailwind configuration

## Routing Approach
The project uses React Router DOM for all routing functionality, **not** Next.js App Router.
- Folder structure mimics Next.js-style organization (e.g., `/creator/[id]/`, `/projects/[id]/applicants/`)
- All routes are explicitly defined in router config (App.tsx or routes.ts)
- No implicit file-based routing logic is used
- Route parameters like `:id` or `:creatorId` are explicitly defined using React Router DOM's `<Route path="...">` format
- All links and route declarations must match the folder structure but use React Router DOM syntax

## Claude CLI Instructions
When working with Claude CLI on this project:
- Help implement new React components following the shadcn/ui and custom design system
- Assist with TypeScript type definitions and interfaces
- Review and refactor code for best practices
- Suggest optimizations for performance
- Help debug issues with components and styling
- Generate test cases for components and functions
- Help build feature-based folder structures
- Implement responsive design with Tailwind
- Create creator and property manager dashboard interfaces
- Implement booking and project posting workflows
- Develop content discovery and filtering mechanisms
- Build notification and toast alert systems
- Ensure route declarations follow React Router DOM patterns while maintaining the folder structure
- Create consistent routing patterns using React Router hooks (useParams, useNavigate)

## Coding Style Preferences
- Use functional components with hooks
- Prefer destructuring for props
- Use named exports for components
- Organize imports alphabetically
- Use Tailwind's utility classes instead of custom CSS when possible
- Implement proper error boundaries and fallbacks
- Follow a modular, token-driven design system
- Maintain clean separation between UI, cards, forms, layout, and dashboard components
- Use explicit route definition with React Router DOM

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgisrael393823) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
