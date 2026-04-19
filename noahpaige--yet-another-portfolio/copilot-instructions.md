## yet-another-portfolio

> - This is a Next.js 15 portfolio website with MDX content management

# Cursor Rules for Yet Another Portfolio

## Project Context

- This is a Next.js 15 portfolio website with MDX content management
- Uses TypeScript, Tailwind CSS, Framer Motion, and Radix UI
- Content is statically generated at build time (no dynamic MDX processing)
- Follows a unified articles system for projects and blogs

## Code Style & Standards

- Use TypeScript strict mode (already configured)
- Follow React 19 best practices with hooks
- Prefer functional components over class components
- Use proper TypeScript interfaces and discriminated unions
- Maintain consistent naming: camelCase for variables, PascalCase for components

## Architecture Patterns

- Keep components small and focused (single responsibility)
- Use proper separation of concerns (components, hooks, lib, types)
- Prefer composition over inheritance
- Implement proper error boundaries and loading states
- Use React.memo() sparingly, only when performance profiling shows benefits

## Content Management

- All content goes in src/articles/ with proper frontmatter
- Use the unified articles system for type safety
- Run content generation scripts before dev/build
- MDX content requires rebuild/redeploy for updates

## Performance & Accessibility

- Implement proper lazy loading for images and components
- Use semantic HTML and ARIA labels
- Optimize animations with Framer Motion
- Implement proper keyboard navigation
- Use intersection observers for performance

## Testing & Quality

- Write meaningful component documentation
- Test content generation scripts regularly
- Validate MDX frontmatter schemas
- Check accessibility compliance
- Monitor bundle size and performance

## AI Assistant Behavior

- **PIPE Method Integration**: Before starting to solve the problem, ask if the user wants to use the PIPE method (Planning, Implementation, Progress tracking, Evaluation). This structured approach ensures thorough problem-solving with proper documentation. The PIPE method involves: 1) **Planning**: Create detailed step-by-step implementation plans with explicit scope boundaries and technical requirements, 2) **Implementation**: Execute the plan step-by-step with AI assistance while monitoring for deviations, 3) **Progress Tracking**: Document what was actually implemented vs. planned, including any changes or issues encountered, 4) **Evaluation**: Create comprehensive feature documentation for future reference and AI onboarding. Always suggest creating documentation files in the `documentation/` folder and use feature branches for organization.
- When suggesting changes, explain the reasoning
- Provide ranked lists of possible solutions
- Focus on workspace improvements over code changes
- Suggest incremental improvements rather than rewrites
- Always consider the build-time content generation workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noahpaige) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
