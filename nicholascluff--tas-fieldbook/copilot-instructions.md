## tas-fieldbook

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start development server
npm run dev

# Start development server with auto-open browser
npm run dev -- --open

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking and validation
npm run check

# Type checking in watch mode
npm run check:watch

# Lint and format code
npm run lint

# Format code
npm run format
```

## Project Architecture

**TasFieldbook** - A Supabase-backed digital fieldbook for Tasmanian land surveyors.

This is a SvelteKit application using:
- **Svelte 5** with the new runes system (`$props()`, `$state()`, etc.)
- **TypeScript** with strict configuration
- **Supabase** for backend, auth, and file storage
- **TailwindCSS v4** with forms and typography plugins
- **Progressive Web App (PWA)** for offline functionality
- **Leaflet with svelte-leafletjs** for interactive mapping

### Application Domain

- **User Model**: Organization-based collaboration system for surveying professionals
- **Organizations**: Companies, firms, or businesses that users can join and collaborate within
- **Project Ownership**: Flexible - any user can own projects, supervision is project-specific
- **Project Phases**: Setup → Fieldwork → Review (with optional supervision)
- **Supervision Model**: Flexible per-project supervision (full, review-only, or advisory)
- **Core Features**: Search analysis, digital diary, map markup, photo management, organization management, flexible invitations
- **Target Users**: Land surveyors in Tasmania working independently, in organizations, or with cross-organization supervision

### Key Architectural Elements

- **Mobile First Development** - Optimized for field use on tablets/phones
- **Offline-First Architecture** - Service workers and IndexedDB for field connectivity
- **Flexible Access Control** - Organization-based permissions with optional supervision workflows
- **Project Ownership Model** - Any user can create and manage projects with flexible visibility settings (private, organization, public)
- **File-Based Routing**: Routes organized by user type and project phase
  - `(auth)/` - Login/register flows
  - `(app)/` - Protected application routes
  - `projects/[id]/` - Project-specific features (search, diary, map, photos, settings, invitations)
  - `invitations/` - Manage project and organization invitations
  - `organizations/` - Organization management and membership
- **Component Organization**:
  - `$lib/components/` - Organized by feature (auth, diary, map, photos, projects, organizations)
  - `$lib/services/` - Supabase API integrations (includes organizations, invitations, permissions, supervision)
  - `$lib/stores/` - Global state management
  - `$lib/types/` - Database and API types with organization support

### Supabase Integration

- **Authentication**: Built-in auth with organization-based permissions
- **Database**: PostgreSQL with Row Level Security (RLS) supporting flexible organization access
- **Storage**: File uploads for PDFs and photos
- **Real-time**: Live updates for organization and project collaboration

### Development Guidelines

- Follow mobile-first responsive design principles
- Implement offline functionality for all field features
- Use Supabase RLS policies for flexible organization and project-based data security
- Support independent practitioners, organization-based workflows, and cross-organization supervision
- Implement organization management, invitation, and permission systems for collaboration
- Optimize images and files for mobile data usage
- Test on actual mobile devices in field conditions

### Configuration Notes

- Environment variables for Supabase connection in `.env`
- TypeScript config extends `.svelte-kit/tsconfig.json` automatically
- ESLint configured for TypeScript + Svelte with Prettier integration
- PWA configuration in `vite.config.ts`
- Service worker setup for offline functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NicholasCluff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NicholasCluff)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
