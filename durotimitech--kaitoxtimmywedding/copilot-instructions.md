## kaitoxtimmywedding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Development

```bash
npm run dev        # Start development server with Turbopack
npm run build      # Build for production
npm run start      # Start production server
npm run preview    # Preview production build locally
```

### Code Quality

```bash
npm run lint       # Check for linting issues
npm run lint:fix   # Auto-fix linting issues
npm run type-check # Check TypeScript types
npm run format     # Format code with Prettier
```

### Testing

```bash
npm test           # Run Jest tests
npm run test:watch # Run tests in watch mode
```

## High-Level Architecture

### Tech Stack

- **Framework**: Next.js 15 with App Router and Turbopack
- **Language**: TypeScript
- **Styling**: Styled Components + Tailwind CSS
- **UI Components**: ShadCN UI
- **Database**: Supabase (PostgreSQL with real-time subscriptions)
- **Authentication**: Supabase Auth
- **State Management**: React hooks and context

### Project Structure

```
/app                    # Next.js app directory
  /(auth-pages)        # Protected routes requiring authentication
    /admin             # Admin seating chart management
    /protected         # Protected info for guests
  /(public-pages)      # Public routes
    /                  # Home page with wedding details
    /gallery           # Photo gallery
    /guestbook         # Real-time guestbook
    /rsvp              # RSVP form
    /songs             # Song request form
    /success           # Success confirmation pages
  /api                 # API routes for form submissions
/components            # React components
  /admin              # Admin-specific components (seating chart)
  /gallery            # Gallery components
  /guestbook          # Guestbook components
  /ui                 # ShadCN UI components
/lib                  # Utilities and configurations
  /supabase           # Supabase client and types
/public               # Static assets and images
/styles               # Global styles
```

### Key Architectural Patterns

1. **Authentication Flow**:
   - Uses Supabase Auth with middleware protection
   - Protected routes under `/(auth-pages)` require authentication
   - Auth middleware in `/middleware.ts`

2. **Real-time Features**:
   - Guestbook uses Supabase real-time subscriptions
   - Updates are pushed to all connected clients instantly

3. **Form Handling**:
   - API routes handle form submissions (RSVP, songs, guestbook)
   - Server-side validation before database insertion
   - Success pages for user feedback

4. **Admin Features**:
   - Drag-and-drop seating chart management
   - Protected by authentication
   - Real-time updates to seating arrangements

5. **Styling Strategy**:
   - Tailwind for utility classes
   - Styled Components for complex component styling
   - ShadCN UI for consistent, accessible components

### Environment Variables

Required in `.env.local`:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`

### Database Schema

Key tables in Supabase:

- `rsvp` - Guest RSVP responses
- `songs` - Song requests
- `guestbook` - Guestbook entries
- `seats` - Seating chart data
- `tables` - Table arrangements

### Development Notes

- Pre-commit hooks run linting and type checking
- Turbopack provides fast HMR in development
- TypeScript strict mode is enabled
- ESLint configured with Next.js best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/durotimitech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
