## astropieter

> Purpose: General marketing as a person of substance and a highly competent professional

# Personal Brand & Purpose
Purpose: General marketing as a person of substance and a highly competent professional

## Brand Identity
- Primary focus: Software engineer and mathematician
- Key differentiator: Applied mathematics and production software expertise
- Personal interests: 
  * Skiing and snowboarding
  * Geopolitics
  * Physical fitness
  * Spiritual growth

# Design System Implementation

## Core Principles
- All styling MUST use CSS variables from src/styles/global.css
- Never hardcode values - always use or create variables
- Maintain consistency across all pages and components
- Ensure responsive design works on all devices
- Color Usage Rules:
  * Only interactive elements (links, buttons) should use the accent blue color (var(--accent))
  * Non-interactive elements should use the non-link text color (var(--non-link-text))
  * Tags, labels, and other UI elements should use appropriate semantic colors (success, warning, etc.)
  * Never use the accent blue for non-interactive elements

## Implementation Process
For any new component or page:
1. Check global.css first for existing variables and patterns
2. Look in src/components/ for similar components to reuse
3. If new styles needed:
   - Add variables to global.css
   - Document the purpose of new variables
   - Follow existing naming conventions

## Component Guidelines
### Links & Interactive Elements
- Use link styles from global.css
- Maintain consistent hover/active states
- External links: open in new tab with rel="noopener noreferrer"
- Follow established behavior patterns

### Responsive Design
- Use existing breakpoint variables
- Test on desktop, tablet, and mobile
- Follow established responsive patterns
- Ensure readability at all screen sizes

# Content & Structure Guidelines

## Landing Page
Reference: https://dileeplearning.github.io/
- Clean, professional layout
- Left: Professional headshot (circular/rounded)
- Right: Concise but comprehensive bio
- Sections to include:
  * Brief introduction
  * Background/experience
  * Current work
  * Interests
  * Education
- Include relevant work/company links
- Add social media/contact info

## Projects Page
Each project should highlight:
- Catchy headline
- Technologies used
- GitHub link ("Source Code")
- Live demo link if available ("See Live")
- Press coverage if any
- Technical deep dive explaining technology choices

## General Content Rules
- Avoid redundant headers (e.g., no "About Me" when menu has "About")
- Use strategic highlighting for current section
- Optimize for quick scanning with clear hierarchy
- Make deep-dive content easily accessible
- Keep animations minimal and quick (max 200ms)
- Show personality while maintaining professionalism
- Focus on specific achievements and affiliations

# Technical Requirements
- Responsive design for all devices
- Clean typography with good spacing
- Professional color scheme (blues, grays, whites)
- Academic/professional but approachable tone
- Optimize for both quick scanning and deep dives
- Maintain consistent navigation and information architecture

- very important: the entire website should be responsive, and work well for desktop, smartphone, and tablets.

lessons from observing other's personal pages:
- bio landing page should looks liek this: 
[https://dileeplearning.github.io/](https://dileeplearning.github.io/)
- 
- clean, professional layout with photo on left, bio content on right
- photo should be circular/rounded and professional headshot style
- bio should be concise but comprehensive, highlighting key achievements
- include sections for: brief intro, background/experience, current work, interests, education
- use clean typography with good spacing and hierarchy
- include links to relevant work/companies mentioned
- layout should be responsive and work well on different screen sizes
- color scheme should be professional (likely blues, grays, whites)
- include social media links/contact info in a clean way
- overall feel should be academic/professional but approachable
- content should establish credibility through specific achievements and affiliations


- projects page should highliht: a catchy headline, technologis used, a link to github, highlight any available press, 

somewhere, perhaps on github , or on a projects details page, we should highlight WHY particular technologies were used. I.e. a technical deep dive


- avoid superfluous verbiage liek a "About ME" header when the menu item already is "About". instead , use strategic highlighting to indicate where in the information architecture we currently are.

- key aspects to highlight: applied mathematics and production software (in contrast to being a research engineer and not understanding production.)

- optimize the entire website for someone who is quickly skimming and only with a small probability doing a deep dive. The grand overview should be easy to sense, and a deep dive made easily accessible. for example , no animations, only if very very quick.

- project items: should have a link to "source code" (GitHub) and "See live" (if available)

- website should be less corporate and anonymous, and show a bit more personality. 
You are an expert Astro.js developer specializing in responsive personal websites. When I ask for help with my personal website, please follow these guidelines:

## Core Principles
- Always use mobile-first responsive design with min-width media queries
- Prioritize performance and accessibility
- Use Astro's built-in features (scoped CSS, Image component, etc.)
- Focus on clean, maintainable code

## Responsive Design Patterns
- Mobile breakpoint: 375px base
- Tablet breakpoint: 768px min-width
- Desktop breakpoint: 1024px min-width
- Use CSS Grid and Flexbox for layouts
- Implement fluid typography with clamp()
- Use CSS custom properties for consistent spacing

## Code Style
- Write mobile-first CSS within <style> tags in .astro files
- Use semantic HTML5 elements
- Implement proper ARIA labels for accessibility
- Use Astro's Image component for optimized images
- Keep styles scoped to components

## Key Responsive Patterns to Include
1. Flexible grid systems (1 col mobile → 2 col tablet → 3 col desktop)
2. Responsive navigation (hamburger mobile → horizontal desktop)
3. Fluid typography using clamp()
4. Optimized images with proper sizing
5. Touch-friendly button sizes (44px minimum)

## Performance Optimizations
- Use lazy loading for below-the-fold content
- Implement proper image optimization
- Minimize CSS and use critical path optimization
- Leverage Astro's partial hydration

When providing code examples, always include:
- Complete .astro component structure
- Mobile-first CSS with breakpoints
- Accessibility considerations
- Performance optimizations
- Brief explanation of the approach

Focus on creating beautiful, fast, and accessible personal websites that work perfectly across all devices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pieteradejong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
