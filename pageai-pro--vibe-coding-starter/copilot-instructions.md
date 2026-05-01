## landing-components

> When working on public pages under /app: landing page, features, contact, etc.


# Landing Page Components Rule

This rule explains how to use landing page components.

When building public-facing pages (e.g., landing, features, contact) under `/app`, use components from the `@landing` module. These components are designed for rapid, consistent, and visually appealing landing page development.

## Component Source

- Import all landing page components from [/components/landing](mdc:shipixen/packages/website/components/landing)
- Use shared UI elements (e.g., `Button`) from [/components/shared/ui](mdc:shipixen/packages/website/components/shared/ui).
- Use icons from `lucide-react` for consistent iconography.
- Use `Image` from `next/image` for optimized images.

## Documentation

- Refer to the official documentation for usage and props: [Landing Page Components Documentation](https:/shipixen.com/boilerplate-documentation/landing-page-components)

## Example Structure

A typical landing page should:
- Compose sections using the following components:
  - `LandingHeader`, `LandingHeaderMenuItem`
  - `LandingPrimaryImageCtaSection`
  - `LandingProductFeaturesGrid`, `LandingProductFeature`
  - `LandingFeatureList`
  - `LandingProductTourSection`, `LandingProductTourList`, `LandingProductTourTrigger`, `LandingProductTourContent`
  - `LandingTestimonialGrid`
  - `LandingPricingSection`, `LandingPricingPlan`
  - `LandingFaqCollapsibleSection`
  - `LandingSaleCtaSection`
  - `LandingFooter`, `LandingFooterColumn`, `LandingFooterLink`
  - `LandingBandSection`
- Use arrays for feature, benefit, testimonial, and FAQ items, mapping over them to render lists.
- Use Tailwind CSS for layout and spacing, following the project's [tailwind-styling.mdc](mdc:shipixen/.cursor/rules/tailwind-styling.mdc)
- @tail
- Use a mobile-first, responsive approach.
- Implement dark mode support

## Best Practices

- Use only the documented props for each component.
- Compose sections in a logical, user-friendly order.
- Follow the project's code style, React, and Next.js best practices [nextjs.mdc](mdc:shipixen/.cursor/rules/nextjs.mdc).

For more details, always refer to the [Landing Page Components Documentation](https:/shipixen.com/boilerplate-documentation/landing-page-components).

---
> Source: [PageAI-Pro/vibe-coding-starter](https://github.com/PageAI-Pro/vibe-coding-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
