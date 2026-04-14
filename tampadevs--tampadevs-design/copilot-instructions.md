## tampadevs-design

> This file provides context for AI coding agents working on the Tampa Devs Design System.

# CLAUDE.md - AI Agent Context

This file provides context for AI coding agents working on the Tampa Devs Design System.

## Project Overview

A monorepo design system providing SSR-compatible React components, design tokens, and a Tailwind preset for Tampa Devs web properties.

**Primary consumers:**
- `events.api.tampa.dev` - React + Tailwind v4
- `tampadevs.com` - Eleventy + SCSS

## Quick Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build all packages (run after changes)
pnpm test             # Run tests
pnpm storybook        # Start Storybook at localhost:6007
pnpm build:storybook  # Build static Storybook
```

## Package Structure

| Package | Path | Purpose |
|---------|------|---------|
| `@tampadevs/tokens` | `packages/tokens/` | CSS custom properties, SCSS variables, JSON tokens |
| `@tampadevs/react` | `packages/react/` | SSR-compatible React components |
| `@tampadevs/tailwind-preset` | `packages/tailwind-preset/` | Tailwind v4 theme preset |
| `@tampadevs/docs` | `apps/docs/` | Storybook documentation |

## Component Files

All components are in `packages/react/src/components/`:

| Component | File | Description |
|-----------|------|-------------|
| `<Button>` | `Button.tsx` | Button with primary/secondary/ghost/danger variants |
| `<EventCard>` | `EventCard.tsx` | Event display card with default/compact/featured variants |
| `<NewsletterSignup>` | `NewsletterSignup.tsx` | Email signup form |
| `<Header>` | `Header.tsx` | Site header with navigation |
| `<Footer>` | `Footer.tsx` | Site footer |
| `<Calendar>` | `Calendar.tsx` | Month calendar with events |
| `<Carousel>` | `Carousel.tsx` | Horizontal scrolling container |
| `<ImageCarousel>` | `ImageCarousel.tsx` | Image slideshow |
| `<VideoEmbed>` | `VideoEmbed.tsx` | YouTube/Vimeo embed |
| `<VideoHero>` | `VideoHero.tsx` | Hero section with video background |
| `<OpenCollective>` | `OpenCollective.tsx` | OpenCollective/GitHub backers display |
| `<SponsorGrid>` | `SponsorGrid.tsx` | Sponsor logos grid |
| `<SponsorCard>` | `SponsorCard.tsx` | Individual sponsor card |
| `<PersonCard>` | `PersonCard.tsx` | Team member card |
| `<PersonTable>` | `PersonTable.tsx` | Team member table |
| `<Table>` | `Table.tsx` | Generic data table |
| `<Avatar>` | `Avatar.tsx` | User avatar |
| `<Icon>` | `Icon.tsx` | Icon library |
| `<Logo>` | `Logo.tsx` | Tampa Devs logo |
| `<Logo3d>` | `Logo3d.tsx` | 3D animated logo (requires Three.js) |
| `<LogoMarquee>` | `LogoMarquee.tsx` | Scrolling logos |
| `<Image>` | `Image.tsx` | Responsive image with loading states |
| `<ImageText>` | `ImageText.tsx` | Image with text overlay |
| `<CtaSection>` | `CtaSection.tsx` | Call-to-action section |
| `<PromoSection>` | `PromoSection.tsx` | Promotional content section |

## Creating a New Component

1. Create `packages/react/src/components/<Name>.tsx`
2. Export from `packages/react/src/index.ts`
3. Create story at `apps/docs/stories/<Name>.stories.tsx`
4. Create test at `packages/react/src/components/<Name>.test.tsx`
5. Run `pnpm build` to compile

### Component Template

```tsx
'use client';

import { useState } from 'react';
import clsx from 'clsx';

export interface ComponentProps {
  /** Component title */
  title?: string;
  /** Additional CSS class */
  className?: string;
  /** Child content */
  children?: React.ReactNode;
}

export function Component({
  title = '',
  className,
  children,
}: ComponentProps) {
  return (
    <>
      <div className={clsx('td-component', className)}>
        {title && <h3 className="td-component__title">{title}</h3>}
        <div className="td-component__content">{children}</div>
      </div>
      <style>{styles}</style>
    </>
  );
}

const styles = `
  .td-component {
    /* Glass morphism - signature style */
    background: rgba(28, 36, 56, 0.8);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 0.75rem;
    padding: 1.5rem;
  }
`;
```

## Design Tokens Reference

See `DESIGN_GUIDELINES.md` for full specification. Key values:

### Colors
```css
--td-color-navy: #1C2438       /* Primary background */
--td-color-navy-light: #2B3447 /* Cards, elevated surfaces */
--td-color-coral: #E85A4F      /* Accent, CTAs */
--td-color-coral-light: #F07167
--td-color-sand: #B6A991       /* Secondary accent */
```

### Glass Morphism (Signature Style)
```css
background: rgba(28, 36, 56, 0.8);
backdrop-filter: blur(12px);
border: 1px solid rgba(255, 255, 255, 0.05);
box-shadow: inset 0 1px 1px 0 rgba(255, 255, 255, 0.05),
            0 8px 24px -4px rgba(0, 0, 0, 0.15);
```

### Spacing (4px grid)
```css
--td-space-1: 0.25rem  /* 4px */
--td-space-2: 0.5rem   /* 8px */
--td-space-4: 1rem     /* 16px */
--td-space-6: 1.5rem   /* 24px */
```

### Border Radius
```css
--td-radius-md: 0.375rem  /* Buttons */
--td-radius-lg: 0.5rem    /* Cards */
--td-radius-xl: 0.75rem   /* Large cards */
```

## Testing

Tests use Vitest with React Testing Library:

```bash
cd packages/react
pnpm test          # Run once
pnpm test:watch    # Watch mode
```

Test file location: `packages/react/src/components/<Name>.test.tsx`

### Test Template

```tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { Component } from './Component';

describe('Component', () => {
  it('renders with default props', () => {
    render(<Component>Test content</Component>);
    expect(screen.getByText('Test content')).toBeInTheDocument();
  });

  it('renders title when provided', () => {
    render(<Component title="Test Title">Content</Component>);
    expect(screen.getByText('Test Title')).toBeInTheDocument();
  });
});
```

## Storybook

Stories are in `apps/docs/stories/`:

```bash
pnpm storybook       # Dev server at localhost:6007
pnpm build:storybook # Build static site
```

### Story Template

```tsx
import type { Meta, StoryObj } from '@storybook/react-vite';
import { Component } from '@tampadevs/react';

const meta: Meta<typeof Component> = {
  title: 'Components/Component',
  component: Component,
  tags: ['autodocs'],
  argTypes: {
    title: { control: 'text' },
  },
  args: {
    title: 'Example',
  },
};

export default meta;
type Story = StoryObj<typeof Component>;

export const Default: Story = {};
```

## CI/CD

| Workflow | Trigger | Action |
|----------|---------|--------|
| `ci.yml` | PR, push to main/master | Build, test, build Storybook |
| `publish.yml` | Release, manual | Publish to GitHub Packages |
| `deploy-storybook.yml` | Push to main/master | Deploy Storybook to Cloudflare Pages |

## Common Tasks

### Add a property to existing component

1. Add prop to interface in component file
2. Update Storybook argTypes if applicable
3. Run `pnpm build`

### Fix a styling issue

1. Find component in `packages/react/src/components/`
2. Modify the `styles` constant
3. Check Storybook for visual verification
4. Run `pnpm build`

### Add external API integration

See `OpenCollective.tsx` for pattern with:
- API fetching with useEffect
- localStorage caching with TTL
- Loading/error states
- Bot filtering for GitHub data

## Important Patterns

### Event Handlers
```tsx
interface Props {
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

function Component({ onClick }: Props) {
  return <button onClick={onClick}>Click me</button>;
}
```

### Conditional Rendering
```tsx
function Component({ loading, error, data }: Props) {
  if (loading) return <div className="loading">Loading...</div>;
  if (error) return <div className="error">{error}</div>;
  if (!data) return null;
  return <div className="content">{/* render data */}</div>;
}
```

### SSR Compatibility
All components use inline `<style>` tags for SSR compatibility:
```tsx
return (
  <>
    <div className="td-component">{children}</div>
    <style>{styles}</style>
  </>
);
```

## Dependencies

- **React** v18/19 - UI library
- **clsx** - Conditional class names
- **Style Dictionary** - Token generation
- **Vite** - Build tool
- **Turborepo** - Monorepo orchestration
- **Vitest** - Testing
- **React Testing Library** - Component testing
- **Storybook** v10 - Documentation

## Files to Reference

| Need | File |
|------|------|
| Visual specs | `DESIGN_GUIDELINES.md` |
| Build config | `turbo.json`, `packages/*/vite.config.ts` |
| Package exports | `packages/*/package.json` |
| Component examples | Any file in `packages/react/src/components/` |
| Story examples | Any file in `apps/docs/stories/` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TampaDevs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TampaDevs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
