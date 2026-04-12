## mavericksedge

> - Use functional components with hooks

# Frontend Development Standards - Mavericks Edge

## React Guidelines
- Use functional components with hooks
- Follow the structure in [client/src/App.tsx](mdc:client/src/App.tsx)
- Use TypeScript for all components
- Implement proper prop interfaces

## Component Architecture
- Use the component structure in [client/src/components/](mdc:client/src/components/)
- Follow single responsibility principle
- Use composition over inheritance
- Implement proper error boundaries

## Key Frontend Components
### Layout System
- Responsive layout with header, footer, and page transitions
- Use Framer Motion for smooth page transitions
- Implement proper loading states

### Service Sections
- Modular sections for different service offerings (web development, marketing, AI integration)
- Use consistent styling with Tailwind CSS
- Implement proper responsive design

### Contact Forms
- Form handling with validation and submission to backend
- Use React Hook Form with Zod validation
- Implement proper error handling and success states

### SEO Components
- Meta tags, structured data, and local SEO optimization
- Use React Helmet for dynamic meta tags
- Implement proper Open Graph and Twitter Card meta tags

## State Management
- Use React Query (TanStack Query) for server state management
- Use local state with `useState` for component state
- Use `useContext` for global state when needed
- Implement proper loading and error states

## Routing
- Use Wouter for client-side routing
- Implement proper route guards
- Handle 404 pages gracefully
- Use lazy loading for route components

## Form Handling
- Use React Hook Form for form management
- Implement Zod validation schemas
- Use proper form error handling
- Follow accessibility guidelines

## UI Components
- Use Radix UI primitives for complex components
- Follow shadcn/ui patterns for component variants
- Use Tailwind CSS for styling
- Implement proper keyboard navigation

## Performance Optimization
- Use `React.memo` for expensive components
- Implement proper `useCallback` and `useMemo` usage
- Use lazy loading for large components
- Optimize bundle size with code splitting
- Implement image optimization with Sharp

## Accessibility
- Use semantic HTML elements
- Implement proper ARIA labels
- Ensure keyboard navigation works
- Test with screen readers

## Asset Management
- Use the asset structure in [client/src/assets/](mdc:client/src/assets/)
- Optimize images and media files
- Use proper file naming conventions
- Implement proper loading states for assets

## Animation Guidelines
- Use Framer Motion for complex animations
- Implement smooth page transitions
- Use GSAP for advanced animations when needed
- Follow performance best practices for animations

description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goodguyben)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/goodguyben)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
