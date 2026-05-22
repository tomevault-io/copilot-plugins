## code-asset-salesforce

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Salesforce LWC (Lightning Web Components) project targeting **Experience Cloud portals** built on LWR (Lightning Web Runtime). API version 66.0. Components are written in TypeScript and styled with SLDS utilities and Experience Cloud design tokens.

## Commands

```bash
# TypeScript compilation (watch mode)

# Lint LWC and Aura components
npm run lint

# Format code
npm run prettier

# Validate deployment (replace alias)
sf project deploy validate --manifest manifest/package.xml --test-level RunLocalTests --target-org <alias>
```

## Architecture

```
force-app/main/default/
  lwc/                        # Lightning Web Components (TypeScript)
  aura/                       # Legacy Aura components
  digitalExperiences/         # Experience Cloud site config and CSS
  digitalExperienceConfigs/
  networks/
  sites/
manifest/package.xml          # Deployment manifest
types/salesforce.d.ts         # Global Salesforce type declarations
```

### LWC Component Layers

- **Page components** — exposed (`exposed: true`), target `lightningCommunity__Page` / `lightningCommunity__Default`
- **Reusable UI components** — internal (`exposed: false`), consumed by other LWCs
- **Utility components** — stateless, handle a single concern (e.g., icon rendering)

## LWC Standards

### TypeScript

- Use `as const` instead of `enum`
- Use `@ts-ignore` above `@api` / `@wire` / `@track` decorators (TypeScript decorator limitation in LWC)
- Avoid unnecessary `@track` — plain properties with immutable updates trigger reactivity
- Type assertions on DOM queries: `this.template.querySelector<HTMLInputElement>('[data-id="x"]')`

### Templates

- Use modern `lwc:if` / `lwc:elseif` / `lwc:else` — never legacy `if:true` / `if:false`
- Use `data-id` attributes for DOM querying; never query by tag or CSS class

### Form Handling

- Single `handleChange` handler routing by `event.target.name`
- Centralized form state object with spread-based immutable updates
- Native DOM validation APIs only: `checkValidity()`, `reportValidity()`, `setCustomValidity()`
- Expose `checkValidity()` / `reportValidity()` / `setCustomValidity()` as `@api` methods on reusable inputs

### CSS & Styling

- SLDS utility classes first (`slds-m-bottom_large`, `slds-p-around_x-large`, etc.)
- Experience Cloud design tokens for color: `var(--dxp-g-brand)`, `var(--dxp-g-destructive)`, `var(--color-text-subtle)`
- Minimal custom CSS — prefer overriding design tokens first. Only create custom CSS when no SLDS class or design token can satisfy the requirement.

### Lifecycle

- Store DOM references in `renderedCallback()` (not constructor)
- Clean up in `disconnectedCallback()`
- Use optional chaining on all queried elements

## Deployment Manifest

Always update `manifest/package.xml` when creating any new Salesforce component. Add the `<members>` entry under the correct `<name>` type block:

- **LWC** → `LightningComponentBundle`
- **Apex class** → `ApexClass`
- **Apex trigger** → `ApexTrigger`
- **Platform Event** → `CustomObject`
- **Other metadata** → match the appropriate metadata type name

## Tooling Notes

- **ESLint**: Flat config (ESLint 9+) in `eslint.config.js` with separate rule sets for Aura, LWC, and Jest mock files
- **Prettier**: 120-char print width, 2-space indent, Apex uses 4-space indent via plugin override
- **tsconfig**: Lives at `force-app/main/default/lwc/tsconfig.json`; component paths mapped as `c/*`

---
> Source: [namnguyen29/code-asset-salesforce](https://github.com/namnguyen29/code-asset-salesforce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
