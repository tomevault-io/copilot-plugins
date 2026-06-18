## cloudscape-codegen

> Generate Cloudscape Design System React + TypeScript UI code and page scaffolds with Cloudscape-first patterns, accessibility defaults, and production-ready state handling.


# Cloudscape Codegen (OpenCode)

This skill generates Cloudscape-first UI code for React + TypeScript + Vite projects.

## When to use this skill

Use this skill when the user asks to:

- build a new Cloudscape page, flow, or component
- scaffold CRUD pages and forms with Cloudscape components
- create data-heavy views with table/filtering/status patterns
- migrate generic React UI to Cloudscape components
- improve Cloudscape UX consistency and accessibility

## When NOT to use this skill

Do not use this skill when:

- the user is not using Cloudscape and does not want to adopt it
- the task is backend-only or infrastructure-only with no UI deliverable
- the user requests a different design system explicitly

## Default assumptions

Unless the user says otherwise, assume:

- React 18+
- TypeScript
- Vite
- Cloudscape component library is available or can be added
- app routing already exists or can be wired with minimal placeholders

If key requirements are missing, make reasonable defaults and state assumptions clearly.

## Core behavior contract

You must:

1. Prefer official Cloudscape components and patterns over custom HTML/CSS.
2. Produce complete, runnable code snippets (not pseudo code).
3. Include loading, empty, and error states for data views.
4. Apply accessible labels, helper text, and keyboard-safe interactions.
5. Keep responsive behavior intentional for desktop and mobile.
6. Avoid invented props, fake Cloudscape APIs, or unsupported component usage.
7. Adapt to existing project conventions when user code context is provided.

## Cloudscape-first decision rules

- **App/page shell**: Prefer `AppLayout` and `ContentLayout` for full pages.
- **Spacing and grouping**: Prefer `SpaceBetween`, `Container`, and `Header`.
- **Forms**: Use `Form`, `FormField`, `Input`, `Select`, `Textarea`, `Checkbox`, `Button`.
- **Data grids**: Prefer `Table` for structured tabular data, with `Pagination` and filters.
- **Alternative data lists**: Prefer `Cards` for card-like/mobile-friendly summaries.
- **Step flows**: Use `Wizard` for guided multi-step tasks.
- **Status and feedback**: Use `Flashbar`, `StatusIndicator`, and `Alert` where appropriate.
- **Navigation aids**: Use `BreadcrumbGroup` for hierarchical page context.

## Output format

For each response, provide:

1. `Approach` (2-5 bullets): layout and component choices.
2. `Code` (TypeScript/TSX): complete component(s) with imports.
3. `Assumptions` (if any): explicit defaults used.
4. `Follow-ups` (optional): concise next integration steps.

If user asks for edits to existing files, provide minimal diffs and preserve existing conventions.

## Data and state guidance

- Model async states explicitly (`loading`, `error`, `empty`, `success`).
- Keep sample data realistic and clearly marked as placeholder data.
- Do not fabricate production endpoints or auth details unless requested.
- If APIs are unknown, wire clear TODO boundaries without blocking UI composition.

## Accessibility and responsiveness gates

Before finalizing output, verify:

- form controls have visible labels and helpful validation text
- table/list actions are reachable and understandable via keyboard
- feedback states are perceivable and not color-only
- layout remains usable on smaller screens

## Reference-only pattern map

Use these repositories as design and architecture references only. Do not copy files verbatim.

- Root examples repo: `https://github.com/aws-samples/cloudscape-examples`
- Starter baseline: `https://github.com/aws-samples/cloudscape-examples/tree/main/empty-vite`
- Multi-page app patterns: `https://github.com/aws-samples/cloudscape-examples/tree/main/basic-vite`
- Chat interaction patterns: `https://github.com/aws-samples/cloudscape-examples/tree/main/chat-ui-vite`
- Fullstack reference (optional inspiration only): `https://github.com/aws-samples/cloudscape-examples/tree/main/fullstack-vite-cdk-cognito-auth`
- Fullstack chat reference (optional inspiration only): `https://github.com/aws-samples/cloudscape-examples/tree/main/full-stack-chat-ui-cognito-auth`

Rule: infer reusable UI patterns from references, then adapt to the target project structure and requirements.

## Anti-copy and safety rules

- Never reproduce external source files verbatim.
- Never claim generated code is official AWS sample code.
- Never assume AWS account resources, regions, or credentials.
- If user requests production readiness, include validation and test suggestions.

## Quick self-check before sending output

- Uses Cloudscape idioms rather than generic UI wrappers
- Includes necessary imports and compiles in a normal TSX setup
- Handles non-happy-path states
- Assumptions are explicit and minimal
- Output is concise and directly usable

---
> Source: [jimmieego/cloudscape-codegen](https://github.com/jimmieego/cloudscape-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
