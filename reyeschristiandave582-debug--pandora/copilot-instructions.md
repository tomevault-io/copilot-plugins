## pandora

> This project is a high-fidelity recreation of a promotional landing page for a Marshalls Gift Card program. It features a professional, trust-oriented aesthetic with a clean white background, navy blue and crimson accents, and interactive elements like a countdown timer and step-by-step instructions.

## Project Summary
This project is a high-fidelity recreation of a promotional landing page for a Marshalls Gift Card program. It features a professional, trust-oriented aesthetic with a clean white background, navy blue and crimson accents, and interactive elements like a countdown timer and step-by-step instructions.

## Tech Stack
- Framework: Next.js 15 (App Router)
- Styling: Tailwind CSS 4
- Icons: Lucide React
- Animations: Framer Motion (via Tailwind classes), CSS animations (float, shine)

## Architecture
- `src/app/page.tsx`: Main entry point assembling all sections.
- `src/components/sections/`: Contains modular components for different parts of the page:
  - `background-overlay.tsx`: Pattern and floating icon background.
  - `announcement-bar.tsx`: Sticky countdown timer bar.
  - `hero-branding.tsx`: Marshalls logo and $1000 gift card visual.
  - `instruction-steps.tsx`: Glassmorphism step-by-step guide.
  - `cta-button.tsx`: Primary action button with shimmer effects.
  - `footer.tsx`: Social links and legal disclaimers.

## User Preferences
- Reduce "Start Review" button size on mobile (42px height, 220px width, 12px text).
- Use "Marshalls Rewards" terminology.
- Maintain specific sub-headlines on a single line for mobile view.

## Project Guidelines
- No comments unless requested.
- Use functional components.
- Maintain pixel-perfect accuracy with the original design tokens.
- Use `whitespace-nowrap` for key sub-headlines on mobile to prevent awkward wrapping.

## Common Patterns
- Modular section-based development.
- Glassmorphism effects using `backdrop-blur` and semi-transparent borders.
- Custom animations for interactive feedback (shine, float, pulse).

---
> Source: [reyeschristiandave582-debug/Pandora](https://github.com/reyeschristiandave582-debug/Pandora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
