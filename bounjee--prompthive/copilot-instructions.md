## prompthive

> PromptHive is a modern AI prompt enhancement platform built with React, TypeScript, Supabase, and Tailwind CSS. It helps users optimize their AI prompts for various platforms (ChatGPT, Claude, Gemini, etc.) across 7 categories.

# PromptHive - AI Prompt Enhancement Platform

## Project Overview
PromptHive is a modern AI prompt enhancement platform built with React, TypeScript, Supabase, and Tailwind CSS. It helps users optimize their AI prompts for various platforms (ChatGPT, Claude, Gemini, etc.) across 7 categories.

## Tech Stack
- **Frontend**: React 18, TypeScript, Vite
- **UI**: Tailwind CSS, Radix UI, Shadcn/ui
- **Backend**: Supabase (PostgreSQL + Auth + RLS)
- **State**: React Query (TanStack), Context API
- **Auth**: Supabase Auth with email/password + Google OAuth
- **Payments**: Polar integration
- **Icons**: Lucide React

## Code Standards

### File Organization
- Use kebab-case for files and folders
- Group related components in folders
- Keep services separate from components
- Single responsibility principle

### React/TypeScript
- Use TypeScript for all files (.tsx/.ts)
- NO React imports unless using React.* features
- Use arrow function components with TypeScript
- Prefer named exports over default exports (except pages)
- Use strict type checking

### Styling
- Tailwind CSS only - NO CSS modules or styled-components
- Use theme colors: bg-[#121212], text-foreground, border-[#27272a]
- Main accent color: #fde047 (yellow)
- Dark theme by default
- Responsive design with mobile-first approach

### Component Patterns
```typescript
// Good component structure
interface ComponentProps {
  title: string;
  isActive?: boolean;
  children?: React.ReactNode;
}

export const Component = ({ title, isActive = false, children }: ComponentProps) => {
  return (
    <div className="bg-gradient-to-br from-[#09090b] to-[#121212] border-[#27272a]">
      {children}
    </div>
  );
};
```

### API & Services
- Use async/await with proper error handling
- All API calls in services/ folder
- Use React Query for data fetching
- Implement loading states and error boundaries

### Database
- PostgreSQL with Supabase
- Row Level Security (RLS) policies
- Admin functions for privileged access
- Use complete-database-setup.sql for setup

## Project Structure
```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Shadcn/ui components
│   ├── auth/           # Authentication components
│   └── account/        # Account-related components
├── pages/              # Route components
├── services/           # API services
├── hooks/              # Custom React hooks
├── contexts/           # React contexts
├── lib/                # Utilities and configs
├── data/               # Static data
├── types/              # TypeScript types
└── constants/          # App constants
```

## Key Features
1. **7 Prompt Categories**: Coding, UI Design, Image Generation, Video Generation, Learning, Technical Writing, General Tasks
2. **AI Enhancement**: GPT-4 powered prompt optimization
3. **Multi-Platform**: Support for ChatGPT, Claude, Gemini, etc.
4. **User Management**: Authentication, profiles, subscriptions
5. **Admin Panel**: Complete system monitoring and user management
6. **Feedback System**: Rating and comments on enhancements
7. **Prompt History**: Save and manage enhanced prompts

## Development Guidelines

### Performance
- Use React.lazy() for route-based code splitting
- Optimize images and assets
- Implement proper loading states
- Use React Query for caching

### Security
- All API routes protected with RLS
- Admin functions use email whitelist
- Sanitize user inputs
- Use proper authentication guards

### Accessibility
- Semantic HTML elements
- Proper ARIA labels
- Keyboard navigation support
- Screen reader friendly

### Error Handling
- Graceful error boundaries
- User-friendly error messages
- Proper logging with debug utilities
- Fallback UI states

## Commands
```bash
# Development
npm run dev

# Build
npm run build

# Type checking
npm run type-check

# Linting
npm run lint
```

## Admin Panel Features
- Dashboard with KPIs and analytics
- User management and activity monitoring
- Prompt enhancement tracking
- Feedback management
- Subscription and payment oversight
- System monitoring with real-time stats

## Never Do
- Don't use CSS modules or styled-components
- Don't import React unless using React.* features
- Don't create duplicate components
- Don't hard-code API URLs
- Don't skip TypeScript types
- Don't ignore accessibility
- Don't commit sensitive data

## Always Do
- Use TypeScript strict mode
- Implement proper error handling
- Add loading states for async operations
- Use semantic HTML
- Follow component composition patterns
- Write self-documenting code
- Test admin functions thoroughly 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bounjee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
