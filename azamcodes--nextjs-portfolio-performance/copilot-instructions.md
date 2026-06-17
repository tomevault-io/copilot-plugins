## nextjs-portfolio-performance

> This skill focuses on reducing load time, minimizing client-side overhead, and improving real-world performance metrics such as Lighthouse and Core Web Vitals.

Skill: Next.js Portfolio Performance Best Practices
Description

Optimizes performance for Next.js portfolio websites by enforcing production-proven performance best practices.
This skill focuses on reducing load time, minimizing client-side overhead, and improving real-world performance metrics such as Lighthouse and Core Web Vitals.

This is an audit and enforcement skill, not a code generator.

Scope

Image optimization using Next.js primitives

Lazy loading of offscreen components and assets

Preference for static rendering (SSG) where applicable

Avoidance of blocking or unnecessary client-side JavaScript

Performance alignment with Lighthouse and Core Web Vitals

Rules

Use next/image for all non-decorative images

Avoid heavy or unnecessary client-side libraries

Prefer static rendering over dynamic rendering when possible

Lazy-load offscreen components using dynamic imports

Minimize blocking client-side JavaScript on initial load

Non-Goals

Does not handle SEO, metadata, or social sharing

Does not define UI, layout, or visual design

Does not generate or rewrite application code

Example Usage

Prompt:

Optimize performance of a Next.js portfolio page using the installed Next.js Portfolio Performance Best Practices skill.


Expected Output:

Images are optimized using next/image

Offscreen components are lazy-loaded

Static rendering is used where applicable

Blocking or unnecessary client-side code is identified

Clear, actionable performance recommendations are provided

---
> Source: [AzamCodes/nextjs-portfolio-performance](https://github.com/AzamCodes/nextjs-portfolio-performance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
