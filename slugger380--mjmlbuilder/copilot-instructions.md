## mjmlbuilder

> \# Project Instructions

\# Project Instructions



This is a web application for generating, editing, managing, previewing, and exporting MJML email templates.



\## Core rules



\- MJML is the primary source format.

\- HTML must always be generated from MJML.

\- Do not remove the existing MJML-to-HTML pipeline.

\- Keep AI generation, MJML serialization, MJML compilation, persistence, and UI logic separated.

\- Prefer structured template state over fragile MJML string concatenation.

\- Keep the whole app as one unified SaaS-style web platform.

\- Preserve unified template mode switching across Builder, AI Generator, Code Editor, Preview, and Settings.

\- Do not hardcode one company identity.

\- Support multiple brand styles.



\## Skill usage



Use $create-plan before large architecture changes, full refactors, routing redesigns, or template workspace redesigns.



Use $playwright after UI changes to open the local app, inspect the rendered UI, click through the main flows, capture screenshots when useful, and verify that the app works in a real browser.



Use $security-best-practices when touching authentication, file uploads, HTML rendering, MJML/HTML sanitization, API endpoints, template exports, or user-generated content.



Use $openai-docs when implementing or updating OpenAI API integration, AI template generation, prompt construction, model selection, structured outputs, or provider abstraction.



Use $figma-implement-design only when a Figma URL, Figma MCP context, or explicit Figma design is available.



Use $gh-fix-ci when GitHub Actions or PR checks fail.



Use $vercel-deploy only when deployment to Vercel is requested.



\## Main product direction



The app should become a professional internal email template management platform with:



\- Dashboard-style app shell

\- Top navigation

\- Templates overview

\- Template cards with preview thumbnails

\- Context menu actions

\- Visual MJML Builder

\- AI template generator

\- Brand style management

\- MJML code editor

\- Preview/testing mode

\- HTML/MJML/JSON export

\- Save/load template persistence



\## Development workflow



1\. Inspect the existing architecture first.

2\. Identify current routes, state management, MJML generation, MJML compilation, storage, and UI structure.

3\. Plan before making broad changes.

4\. Make changes in reviewable steps.

5\. Keep code modular and typed.

6\. Reuse existing components and utilities where possible.

7\. Run available lint/build/test commands after changes.

8\. Verify the main template workflow in the browser when possible.

---
> Source: [slugger380/MJMLbuilder](https://github.com/slugger380/MJMLbuilder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
