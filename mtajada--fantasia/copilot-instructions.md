## fantasia

> **Fantasia** is undergoing transformation from a children's storytelling application to an adult-oriented erotic content platform. The project generates personalized stories with voice narration and interactive elements, now targeting mature audiences with sophisticated adult themes.


## Project Overview

**Fantasia** is undergoing transformation from a children's storytelling application to an adult-oriented erotic content platform. The project generates personalized stories with voice narration and interactive elements, now targeting mature audiences with sophisticated adult themes.

**Current Version**: 1.1.4  
**Main Branch**: master  
**Project Type**: Single Page Application (SPA)  
**Target Audience**: Adults (18+)  
**Content Focus**: Adult erotic literature and interactive experiences  
**Language**: English (migrating from Spanish)

## Technology Stack

### Frontend
- **React 18.3.1** with TypeScript
- **Vite** for build tooling and development server
- **Tailwind CSS** for styling with custom configuration
- **shadcn/ui** component library (extensive Radix UI components)
- **Framer Motion** for animations and transitions
- **Zustand** for state management
- **React Router DOM** for navigation
- **React Hook Form** with Zod validation
- **TanStack Query** for server state management

### Backend & Services
- **Supabase** as Backend-as-a-Service (BaaS)
  - Authentication and user management
  - PostgreSQL database with Row Level Security (RLS)
  - Edge Functions for serverless compute
  - File storage for audio and images
- **OpenAI** for AI-powered story generation and TTS
- **Google Generative AI** (Gemini) for story generation
- **Stripe** for payment processing and subscriptions
- **ElevenLabs** for voice synthesis (via API)

### Development Tools
- **TypeScript** with strict configuration
- **ESLint** with React and TypeScript rules
- **PostCSS** with Tailwind CSS
- **PM2** for production deployment
- **Bun** as package manager (with npm fallback)

## Project Structure

```
FantasIA/
├── src/
│   ├── components/           # Reusable UI components
│   │   ├── ui/              # shadcn/ui components (~49 files)
│   │   └── [various].tsx    # App-specific components
│   ├── pages/               # Route-based page components
│   ├── services/            # API integration layer
│   │   ├── ai/             # AI service wrappers
│   │   └── [various].ts    # Database and third-party services
│   ├── store/              # Zustand state management
│   │   ├── character/      # Character management
│   │   ├── stories/        # Story-related state
│   │   ├── user/           # User profile and auth
│   │   └── core/           # Store utilities
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utility functions
│   ├── types/              # TypeScript type definitions
│   └── config/             # App configuration
├── supabase/
│   ├── functions/          # Edge Functions
│   ├── migrations/         # Database migrations
│   └── sql-functions/      # Database functions
├── docs/                   # Project documentation
└── public/                 # Static assets
```

## Key Features

### Story Generation
- **Personalized Adult Stories**: AI-generated erotic tales based on character preferences, kinks, and interests
- **Chapter System**: Multi-chapter stories with continuation options
- **Story Format Choice**: Users can choose between 'single' self-contained stories or 'episodic' stories designed for multiple chapters
- **Multiple Genres**: Romance, BDSM, fantasy, contemporary, etc.
- **Character Customization**: Name, gender ('male', 'female', 'non-binary'), and free-text description

### Audio Features
- **Text-to-Speech**: Professional voice narration using OpenAI TTS with sensual voices
- **Multiple Voices**: Different personality-matched voices for adult content
- **Audio Player**: Custom audio player with progress tracking
- **Voice Preview**: Sample voices before selection

### Image Generation
- **Currently Deactivated**: Image generation functionality is temporarily disabled
- **Database Ready**: The schema includes a 'cover_image_url' field in the 'stories' table for future implementation
- **Planned Features**: DALL-E 3 integration for story illustrations, cover images, and character portraits

### Adult Content Features
- **Content Warnings**: Appropriate warnings for different types of adult content
- **Age Verification**: Robust age verification system
- **Customizable Content**: User preferences for content intensity and themes
- **Privacy Controls**: Enhanced privacy features for adult content

### User Management
- **Authentication**: Supabase Auth with email/password
- **Profiles**: User preferences and adult content settings
- **Subscriptions**: Stripe-powered payment system
- **Usage Tracking**: Story and voice credit limits

## Development Workflow

### Local Development
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build:prod

# Preview production build
npm run start:prod
```

### Key Commands
- `npm run dev` - Start development server (localhost:8080)
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run deploy` - Build and start production server

### Environment Setup
Required environment variables:
- `VITE_SUPABASE_URL` - Supabase project URL
- `VITE_SUPABASE_ANON_KEY` - Supabase anonymous key
- `VITE_STRIPE_PUBLISHABLE_KEY` - Stripe publishable key
- `VITE_ELEVENLABS_API_KEY` - ElevenLabs API key

## Architecture Patterns

### State Management
- **Zustand stores** for global state
- **Store separation** by domain (user, character, stories, etc.)
- **Persistence** with localStorage for offline support
- **Sync queue** for offline-first data handling

### Service Layer
- **Abstraction layer** between UI and backend
- **Edge Function wrappers** for AI services
- **Direct database access** with RLS security
- **Error handling** with standardized response format

### Component Architecture
- **Atomic design** with reusable components
- **Page-based routing** with React Router
- **Custom hooks** for shared logic
- **TypeScript** for type safety

## Database Schema

### Main Tables
- `profiles` - User profiles with language settings and adult content preferences (free-text field for tastes, kinks, etc.)
- `characters` - Simplified custom characters with only name, gender, and description
- `stories` - Generated stories with metadata, including story_format ('single' or 'episodic'), flexible genre field, and cover_image_url for future use
- `story_chapters` - Individual story chapters (used for stories marked as 'episodic')
- `audio_files` - Generated audio recordings
- `user_voices` - Voice preferences
- `preset_suggestions` - Story prompt presets for generation

### Security
- **Row Level Security (RLS)** on all tables
- **User-based access control**
- **Secure API key management** in Edge Functions

### SQL Documentation
**IMPORTANT**: `/docs/sql_supabase.sql` is the **canonical reference** for the database schema. This is the definitive migration script that contains:
- Complete table definitions and structure
- All RLS policies and permissions
- Database enums and constraints
- The exact SQL to be executed in Supabase SQL Editor

**Development Guidelines**:
- **Always refer to** `/docs/sql_supabase.sql` for any SQL-related queries or modifications
- **Do NOT modify** this file unless absolutely necessary for new functionality
- This script is designed to be executed in the Supabase SQL Editor to apply changes remotely
- Other SQL files in `/docs/` may be outdated and should be considered legacy

Legacy files (may be outdated):
- `/docs/supabase_tables.sql` - Legacy table definitions
- `/docs/supabase_RLS.sql` - Legacy RLS policies

## Testing

**Note**: Currently no test framework is configured. The project uses:
- Manual testing in development
- TypeScript for compile-time validation
- ESLint for code quality
- Production monitoring for runtime issues

## Deployment

### Production Setup
- **PM2** for process management
- **Nginx** reverse proxy (not included in repo)
- **Environment variables** for configuration
- **Supabase Edge Functions** for serverless compute

### PM2 Configuration
```javascript
// ecosystem.config.cjs
{
  name: "cuenta-cuentos",
  script: "npm",
  args: "run start:prod",
  env: {
    NODE_ENV: "production",
    PORT: "8080"
  }
}
```

## Development Guidelines

### Transformation Rules
1. **Think First**: Read codebase for relevant files, write plan to tasks/todo.md
2. **Check Before Working**: Always verify plan with user before implementation
3. **High-Level Updates**: Give brief explanations of changes made
4. **Simple Changes**: Make every task as simple as possible, minimal code impact
5. **Review Documentation**: Add review section to todo.md with change summary

### Project Transformation Context
- **Content Migration**: Transform from children's stories to adult erotic content
- **Language Migration**: Gradually change from Spanish to English (new features in English)
- **Architecture Migration**: Replace Zustand local store with direct Supabase queries
- **Simplicity Focus**: Avoid massive or complex changes, every change should be incremental

### Code Style
- **TypeScript strict mode** enabled
- **ESLint** configuration with React rules
- **Consistent naming** (camelCase for JS, snake_case for DB)
- **Component organization** by feature/domain
- **English-first approach** for new functions and components

### Best Practices
- **Prefer editing** existing files over creating new ones
- **Use absolute imports** with `@/` alias
- **Handle errors** gracefully with user feedback
- **Implement loading states** for async operations
- **Follow Tailwind CSS** utility-first approach
- **Direct Supabase integration** for new features (avoid Zustand dependency)

### State Management Migration
- **Legacy**: Zustand stores with localStorage persistence
- **Target**: Direct Supabase queries with real-time subscriptions
- **Approach**: Gradual migration, maintain existing patterns during transition
- **New Features**: Extract data directly from Supabase, don't use local stores

## API Integration

### Edge Functions
- `generate-story` - AI adult story generation with mature themes
- `story-continuation` - Story continuation options for adult narratives
- `generate-audio` - Text-to-speech conversion with sensual voices
- `upload-story-image` - Adult content image generation and storage (Currently Deactivated)
- Stripe functions for payment processing

### External APIs
- **OpenAI** - GPT models and TTS
- **Google Generative AI** - Gemini models
- **Stripe** - Payment processing
- **ElevenLabs** - Voice synthesis

## Common Issues & Solutions

### Authentication
- **Race conditions** handled with auth guards
- **Session management** through Supabase client
- **Redirect handling** for auth callbacks

### Performance
- **Lazy loading** for route components
- **Image optimization** with proper formats
- **Audio streaming** for large files
- **State persistence** to avoid refetching

### Offline Support
- **Sync queue** for failed operations
- **localStorage persistence** for critical data
- **Network detection** for connectivity changes

## Implementation Guidelines

### Adult Content Considerations
- **Content Moderation**: Implement appropriate content filtering
- **Privacy First**: Enhanced privacy features for sensitive content
- **Age Verification**: Robust verification system
- **Content Warnings**: Clear labeling of content types and intensity

For detailed implementation guides, see the `/docs` directory and `/tasks/todo.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mtajada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
