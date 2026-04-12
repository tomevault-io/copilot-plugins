## whatrack

> - **Primary**: `pnpm` with workspace support

# Development Workflow & Tooling Rules

## 🛠️ Monorepo Management

### Package Manager
- **Primary**: `pnpm` with workspace support
- **Lock File**: [pnpm-lock.yaml](mdc:pnpm-lock.yaml) - commit this file
- **Workspace Config**: [pnpm-workspace.yaml](mdc:pnpm-workspace.yaml)

### Build System
- **Turborepo**: Task orchestration and caching
- **Config**: [turbo.json](mdc:turbo.json) - defines task dependencies
- **Commands**: Use `pnpm` commands from [package.json](mdc:package.json)

## 📝 Development Commands

### Quick Start
```bash
# Install all dependencies
pnpm install

# Development servers
pnpm dev                # Start all packages + apps in watch mode
pnpm dev:native         # Start mobile app (Expo)
pnpm dev:web           # Start web app (Vite) 
pnpm dev:server        # Start backend API (Hono)
pnpm dev:shared        # Start shared package in watch mode

# Recommended for active development
pnpm dev:shared         # Terminal 1: Start shared package watcher
pnpm dev:server         # Terminal 2: Start your app
```

### Build & Quality
```bash
# Build all packages and apps for production
pnpm build
pnpm build:shared       # Build shared package only

# Code quality and formatting  
pnpm check              # Format and lint with Biome
pnpm check-types        # TypeScript validation across all apps

# Database operations (server only)
pnpm db:push            # Push schema changes to database
pnpm db:studio          # Open Drizzle Studio database GUI
```

## 🔧 Code Quality Tools

### Biome Configuration
- **Config**: [biome.json](mdc:biome.json) - formatting and linting rules
- **Usage**: Automatically formats and lints TypeScript/JavaScript
- **Integration**: Works with VS Code and CI/CD

### TypeScript Configuration
- **Shared Config**: [tsconfig.json](mdc:tsconfig.json) in workspace root
- **Strict Mode**: Enabled across all apps for type safety
- **Path Mapping**: Configured for each app's specific needs

### Git Hooks & Pre-commit
```bash
# Recommended pre-commit hook
#!/bin/sh
pnpm check              # Format and lint
pnpm check-types        # Type checking
```

## 📱 Mobile Development

### Expo Workflow
```bash
# Navigate to mobile app
cd apps/native

# Start development server
pnpm start              # Same as expo start

# Platform-specific builds
pnpm android           # Build for Android
pnpm ios               # Build for iOS
pnpm web               # Test in web browser
```

### Testing Mobile App
- **Physical Device**: Install Expo Go app
- **Simulator**: Use Android Studio or Xcode simulators
- **Web Browser**: Quick testing without device setup

## 🌐 Web Development

### Vite Development
```bash
# Navigate to web app
cd apps/web

# Start development server (hot reload)
pnpm dev

# Preview production build
pnpm preview

# Build for production
pnpm build
```

### shadcn/ui Components
```bash
# Add new shadcn/ui components
npx shadcn@latest add button card input

# Components auto-added to src/components/ui/
# Configuration in components.json
```

## 📦 Shared Package Development

### Working with @whatrack/shared
```bash
# Navigate to shared package
cd packages/shared

# Development (watch mode - rebuilds on changes)
pnpm dev

# One-time build
pnpm build

# Clean build artifacts
pnpm clean

# Type checking
pnpm typecheck
```

### Development Workflow
1. **Start shared package in watch mode**: `pnpm dev:shared`
2. **Start your app**: `pnpm dev:server` or `pnpm dev:native`
3. **Make changes to shared utilities** - auto-rebuilds
4. **Test changes in app** - restart app if types changed

### Adding New Shared Utilities
```bash
# 1. Add utility to packages/shared/src/
# 2. Export from packages/shared/src/index.ts
# 3. Import in apps using '@whatrack/shared'
import { newUtility } from '@whatrack/shared';

# Example: Phone utilities
import { normalizePhoneNumber } from '@whatrack/shared/phone';
```

### Best Practices
- Keep shared package **lightweight and focused**
- Only add utilities used by **2+ apps**
- Use **Zod schemas** for validation logic
- Document new utilities in package README
- Prefer **pure functions** over stateful logic

## 🗄️ Server Development

### Database Workflow
```bash
# Navigate to server app
cd apps/server

# Database operations
pnpm db:push            # Push schema to database
pnpm db:studio          # Open database GUI
pnpm db:generate        # Generate migrations

# Development server with hot reload
pnpm dev
```

### Environment Setup
```bash
# .env.local (never commit)
DATABASE_URL="file:./dev.db"
JWT_SECRET="your-development-secret"
```

## 🔄 Version Control

### Git Workflow
```bash
# Feature branch workflow
git checkout -b feature/contact-permissions
git add .
git commit -m "feat: add contact permissions flow"
git push origin feature/contact-permissions

# Pull request to main branch
```

### Commit Convention
```bash
# Use conventional commits
feat: add new feature
fix: fix bug
docs: update documentation
style: formatting changes
refactor: code refactoring
test: add tests
chore: maintenance tasks
```

### Files to Ignore
- `node_modules/` (auto-ignored)
- `.env.local` and `.env` files with secrets
- Platform-specific build outputs
- IDE configuration files

## 🚀 Production Deployment

### Mobile App Deployment
```bash
# Expo Application Services (EAS)
cd apps/native

# Build for app stores
eas build --platform android
eas build --platform ios

# Submit to stores
eas submit --platform android
eas submit --platform ios
```

### Web App Deployment
```bash
# Build optimized production bundle
cd apps/web
pnpm build

# Deploy to Vercel/Netlify/etc.
# Built files in dist/ directory
```

### Server Deployment
```bash
# Build and deploy with SST
cd apps/server
pnpm deploy

# Environment variables needed:
# DATABASE_URL, JWT_SECRET, etc.
```

## 🐛 Debugging & Development

### Mobile Debugging
- **React Native Debugger**: For component inspection
- **Expo DevTools**: Built-in debugging tools
- **Console Logs**: Use `console.log()` for quick debugging
- **Error Overlay**: Expo shows runtime errors in app

### Web Debugging
- **Browser DevTools**: Standard web debugging
- **React DevTools**: Component tree inspection
- **Vite HMR**: Hot module replacement for fast iteration

### API Debugging
- **tRPC Panel**: Interactive API explorer
- **Drizzle Studio**: Database GUI for data inspection
- **Server Logs**: Console output for API requests

## 🧪 Testing Strategy

### Current Status
- **Phase 1**: Manual testing with mock data
- **Phase 2**: Unit tests for API endpoints
- **Phase 3**: E2E tests for critical flows

### Testing Tools (Future)
```bash
# Mobile testing
jest + @testing-library/react-native

# Web testing  
vitest + @testing-library/react

# API testing
vitest + supertest

# E2E testing
playwright or cypress
```

## 📊 Performance Monitoring

### Development Metrics
- **Bundle Size**: Monitor with `pnpm build`
- **Type Checking Speed**: Track `pnpm check-types` time
- **Build Performance**: Turborepo caching effectiveness

### Production Monitoring (Future)
- **Mobile**: Expo Application Services metrics
- **Web**: Core Web Vitals monitoring  
- **API**: Response time and error rate tracking

## 🎯 Development Best Practices

### Code Organization
- **Consistent File Naming**: kebab-case for files, PascalCase for components
- **Clear Imports**: Use absolute imports where possible
- **Type Safety**: Leverage TypeScript strict mode
- **Error Handling**: Graceful error boundaries and fallbacks

### Performance
- **Bundle Splitting**: Automatic with Vite and Expo
- **Code Splitting**: Dynamic imports for large components
- **Caching**: Leverage Turborepo build caching
- **Optimization**: Tree shaking and minification in production

### Security
- **Environment Variables**: Never commit secrets
- **Dependencies**: Regular security audits with `pnpm audit`
- **API Security**: Rate limiting and input validation
- **Mobile Security**: Secure storage for sensitive data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianarchila)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julianarchila)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
