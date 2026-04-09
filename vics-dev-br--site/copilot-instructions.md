## site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VICS (www.vics.dev.br) is an Angular 19 Progressive Web Application (PWA) with Server-Side Rendering (SSR) for a data science/analytics company. The project uses Angular Universal for SEO optimization and includes offline capabilities through service workers.

## Commands

### Development
- `npm start` - Start development server at http://localhost:4200/
- `npm run serve:ssr` - Run the SSR server locally using the built distribution

### Build
- `npm run build` - Build for production with optimization
- `npm run build:prod` - Same as above, explicitly production build
- `npm run build:test` - Build for test environment
- `npm run build:ssr` - Build both client and server bundles for SSR

### Testing & Quality
- `npm test` - Run unit tests with Karma/Jasmine in Chrome (watch mode enabled by default)
- `npm run lint` - Run TSLint with Codelyzer for Angular-specific checks
- `npm run e2e` - Run end-to-end tests with Protractor
- Coverage reports are generated in `./coverage/xavro/` directory
- To run a single test file: `ng test --include='**/path-to-spec.ts'`
- To focus on specific tests: use `fdescribe()` or `fit()` in test files

### Angular CLI
- `ng generate component modules/[module-name]/components/[component-name]` - Generate new component in appropriate module
- `ng generate service modules/[module-name]/services/[service-name]` - Generate new service
- `ng generate module modules/[module-name] --route [route-name] --module app.module` - Generate new lazy-loaded module

## Architecture

### Module Structure
The application follows a modular architecture with lazy-loaded feature modules:

```
src/app/
├── core/           # Core services (BaseService)
├── modules/        # Feature modules (all lazy-loaded)
│   ├── blog/       # Blog functionality with list/detail views
│   ├── home/       # Main module with multiple marketing components
│   ├── landing/    # Landing page module
│   ├── lead/       # Lead generation and contact forms
│   └── privacy_policy/
└── shared/         # Shared components (navbar, footer)
```

### API Configuration
- Production: `https://669n3b1h62.execute-api.us-east-1.amazonaws.com/prod/`
- Development: `http://localhost:3000/`
- Configured in `src/environments/`

### Key Technologies
- Angular 19.2.11 with TypeScript 5.8.3
- Bootstrap 5.3 for styling
- FontAwesome for icons
- Animate.css for animations
- ngx-countup, ngx-scrolltop, ngx-typed-js for UI enhancements

### Deployment
The project is configured for AWS Amplify deployment:
- Node.js 22 runtime
- Build output: `dist/vics/browser`
- SSR server: `dist/vics/server/server.mjs`

### Code Style
- TSLint configuration enforces Angular style guide with Codelyzer
- Single quotes for strings
- 140 character line limit
- Component selectors use `app-` prefix with kebab-case
- Directive selectors use `app` prefix with camelCase
- Semicolons are required
- Console methods (debug, info, time, timeEnd, trace) are banned
- Enforces proper lifecycle interface implementation
- Member ordering: static fields → instance fields → static methods → instance methods

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vics-dev-br)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vics-dev-br)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
