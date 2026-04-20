## robinabdullah-github-io

> - Main layout components in [src/components/layout](mdc:src/components/layout)

# Component Structure Guidelines

## Component Organization
- Main layout components in [src/components/layout](mdc:src/components/layout)
- Page section components in appropriate directories by feature
- Reusable UI components should be isolated and well-documented

## Component Principles
- Use TypeScript for all components
- Follow functional component pattern with React hooks
- Implement proper prop typing with interfaces
- Keep components focused on a single responsibility

## Navigation Components
The [Header.tsx](mdc:src/components/layout/Header.tsx) component manages site navigation and should:
- Maintain active section highlighting
- Provide smooth scrolling to sections
- Be responsive on all devices

## Section Components
Each main section should:
1. Fetch data from portfolio.json
2. Implement proper error handling
3. Use consistent styling approach
4. Be responsive across all device sizes

## Styling Approach
- Use Tailwind CSS for styling
- Maintain consistent color scheme using theme variables
- Implement responsive design using Tailwind breakpoints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robinabdullah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
