## auxla-project

> Mental health awareness and support platform built with React, TypeScript, and Tailwind CSS.

# OCSLAA Project - Auto-Coding Rules

## Project Overview
Mental health awareness and support platform built with React, TypeScript, and Tailwind CSS.

## Core Principles
1. **User-Centric Design**: Prioritize accessibility, empathy, and user comfort
2. **Performance First**: Optimize for fast load times and smooth interactions
3. **Type Safety**: Leverage TypeScript for robust, maintainable code
4. **Responsive Design**: Mobile-first approach with seamless desktop experience
5. **Accessibility**: WCAG 2.1 AA compliance minimum

## Code Style & Standards

### TypeScript
- Use functional components with TypeScript interfaces
- Prefer `interface` over `type` for object shapes
- Use explicit return types for all functions
- Enable strict mode and fix all type errors
- No `any` types unless absolutely necessary (document why)
- Use proper TypeScript generics for reusable components

### React Best Practices
- Use functional components with hooks (no class components)
- Extract custom hooks for reusable logic (prefix with `use`)
- Keep components small and focused (max 200 lines)
- Use React.memo() for expensive re-renders
- Implement proper error boundaries
- Use Suspense and lazy loading for code splitting

### Component Structure
```tsx
// 1. Imports (group by: react, external libs, internal, types, styles)
import { useState, useEffect } from "react";
import { useQuery } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import type { UserData } from "@/types";

// 2. Types/Interfaces
interface ComponentProps {
  title: string;
  onAction: () => void;
}

// 3. Component
export const Component = ({ title, onAction }: ComponentProps) => {
  // 4. Hooks (state, effects, custom hooks)
  const [isOpen, setIsOpen] = useState(false);
  
  // 5. Event handlers
  const handleClick = () => {
    // ...
  };
  
  // 6. Render logic
  return (
    <div>
      {/* JSX */}
    </div>
  );
};
```

### Styling with Tailwind CSS
- Use Tailwind utility classes exclusively
- Follow mobile-first responsive design: `sm:`, `md:`, `lg:`, `xl:`
- Use design system tokens from `index.css` for colors
- Group related classes logically: layout → spacing → colors → typography
- Use `cn()` utility from `lib/utils.ts` for conditional classes
- Create custom components in `components/ui/` for reusable patterns
- Avoid inline styles unless absolutely necessary

### Naming Conventions
- **Components**: PascalCase (`UserProfile.tsx`)
- **Hooks**: camelCase with `use` prefix (`useAuth.ts`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Constants**: SCREAMING_SNAKE_CASE (`API_BASE_URL`)
- **Interfaces**: PascalCase with descriptive names (`UserProfileData`)
- **Types**: PascalCase (`StringOrNumber`)
- **Files**: Match component/export name

### File Organization
```
src/
├── components/
│   ├── ui/           # Reusable UI components (shadcn)
│   ├── features/     # Feature-specific components
│   └── layout/       # Layout components (Header, Footer)
├── pages/            # Page components (route level)
├── hooks/            # Custom React hooks
├── lib/              # Utilities and helpers
├── types/            # TypeScript type definitions
├── services/         # API and external service integrations
├── store/            # State management (if needed)
├── constants/        # Application constants
└── assets/           # Static assets (images, fonts)
```

### State Management
- Use React Query for server state
- Use Context API for simple global state
- Prefer local state over global when possible
- Use Zustand for complex client state (if needed)

### API Integration
- Create service files in `services/` directory
- Use React Query hooks for data fetching
- Implement proper error handling with try/catch
- Add loading and error states to all async operations
- Use environment variables for API endpoints

### Form Handling
- Use React Hook Form for all forms
- Implement Zod schemas for validation
- Show clear, helpful error messages
- Disable submit buttons during submission
- Provide visual feedback for all actions

### Error Handling
- Implement error boundaries for component errors
- Show user-friendly error messages
- Log errors appropriately (console in dev, service in prod)
- Provide recovery actions when possible
- Never expose technical details to users

### Performance
- Lazy load routes and heavy components
- Optimize images (use WebP, proper sizes)
- Implement virtual scrolling for long lists
- Debounce search and input handlers
- Use React.memo() judiciously
- Minimize bundle size (check with bundle analyzer)

### Accessibility
- Use semantic HTML elements
- Add ARIA labels where needed
- Ensure keyboard navigation works
- Maintain color contrast ratios (WCAG AA)
- Add alt text to all images
- Support screen readers
- Focus management for modals and dialogs

### Testing Strategy
- Write unit tests for utilities and hooks
- Component tests for UI components
- Integration tests for user flows
- E2E tests for critical paths
- Aim for 80% code coverage

### Security
- Sanitize all user inputs
- Use HTTPS for all API calls
- Implement proper CORS policies
- Store sensitive data securely
- Validate data on both client and server

### Git Workflow
- Feature branches: `feature/feature-name`
- Bug fixes: `fix/bug-description`
- Hotfixes: `hotfix/issue-description`
- Commit format: `type(scope): description`
  - Types: feat, fix, docs, style, refactor, test, chore
- Keep commits atomic and descriptive

### Documentation
- Add JSDoc comments to complex functions
- Document props with TypeScript interfaces
- Create README for complex features
- Keep API documentation updated
- Document environment variables

### Mental Health Specific Guidelines
- Use inclusive, supportive language
- Avoid triggering terminology
- Ensure crisis resources are always visible
- Make help easily accessible (max 2 clicks)
- Test with mental health professionals
- Consider cognitive load in UI design
- Use calming colors and smooth animations

### Dependencies
- Review before adding new dependencies
- Prefer well-maintained packages
- Check bundle size impact
- Keep dependencies updated regularly
- Document why each dependency is needed

### Code Review Checklist
- [ ] TypeScript types are correct and strict
- [ ] Component is properly tested
- [ ] Accessibility requirements met
- [ ] Responsive on all breakpoints
- [ ] Error handling implemented
- [ ] Loading states added
- [ ] Performance optimized
- [ ] Code is documented
- [ ] Follows naming conventions
- [ ] No console.logs in production code

## Environment Variables
```env
VITE_API_BASE_URL=
VITE_STRIPE_PUBLIC_KEY=
VITE_ANALYTICS_ID=
VITE_SENTRY_DSN=
```

## Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run lint` - Lint code
- `npm test` - Run tests

## Resources
- Design System: See `src/index.css`
- UI Components: shadcn/ui documentation
- Icons: Lucide React
- Tailwind: https://tailwindcss.com/docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaltech21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
