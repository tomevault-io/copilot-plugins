## neptunik

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 Principios Fundamentales

1. Lee archivos completos antes de modificar
2. Implementa funcionalidad real (no dummy)
3. Aplica clean architecture
4. Aplica principios SOLID
5. Los ficheros se nombran con kebab-case
6. Selecciona agentes automáticamente según contexto
7. Evalúa siempre qué agentes de los que dispones pueden ayudarte a cumplir lo que se te pide

## Behavior

1. When defining an action plan, I want that plan to also include the necessary tasks, covering both research and execution. Each task should be assigned to the appropriate expert agent or agents, and prioritized so that tasks which can be carried out in parallel are executed simultaneously to save time.

## Documentation

- **business and product** documents are located in /product/ folder, including the
  knowledger base, business and product decisions, research, and executive
  summaries
  - **development and usage documentation** is located in /docs/ folder

## Project Overview

**Neptunik** - Platform for WhatsApp Cloud API integration with complete onboarding flow.

## Technology Stack

### Core Framework & Language

- **Framework**: Next.js 15.5.2 with App Router
- **Language**: TypeScript 5.9.2
- **Runtime**: Node.js >=18.0.0

### Frontend Stack

- **Styling**: Tailwind CSS 3.4.17
- **State Management**: Redux Toolkit 2.8.2 + React Redux 9.2.0
- **UI Components**: Custom components based on Radix UI primitives
- **Forms**: React Hook Form 7.62.0 with Zod 4.1.5 validation
- **Animation**: Framer Motion 12.23.12
- **Icons**: Lucide React
- **Utilities**: class-variance-authority, tailwind-merge, clsx

### Testing & Quality Assurance

- **Unit Testing**: Jest with Next.js integration
- **Component Testing**: React Testing Library + Jest DOM
- **E2E Testing**: Playwright (Chrome, Firefox, Safari, mobile)
- **Accessibility**: WCAG 2.1 AA compliance testing
- **Coverage**: 80-95% thresholds (Domain: 95%, Application: 90%)

### Media & Export Capabilities

- **GIF Generation**: gif.js + html2canvas
- **Canvas Mocking**: jest-canvas-mock for testing
- **Web Workers**: Background GIF processing

### Analytics & Monitoring

- **Error Tracking**: @sentry/nextjs + @sentry/replay
- **Analytics**: @vercel/analytics + posthog-js
- **Performance**: @vercel/speed-insights + web-vitals 5.1
- **A/B Testing**: Custom implementation with PostHog

### Development Tools

- **Package Manager**: npm (>=8.0.0)
- **Linting**: ESLint with Next.js integration
- **Dev Server**: Turbopack (with Webpack fallback)

## Commands

### Development

```bash
# Recommended for daily use
npm run dev:stable      # Optimized stable development (8GB, GC tuned)
npm run dev:performance # High performance for complex features (12GB, aggressive GC)
npm run dev:memory      # Memory debugging mode (16GB, expose-gc)

# Specialized modes
npm run dev:hot-reload  # Optimized HMR with Fast Refresh
npm run dev:production  # Development with production-like config
npm run dev:webpack     # Webpack fallback when Turbopack fails
npm run dev:fallback    # Minimal config for limited resources

# Debug & Analysis
npm run dev:debug       # Debug mode with detailed Turbopack logging
npm run dev:clean       # Clean cache + stable development
npm run turbo:analyze   # Turbopack performance analysis

# Production
npm run build          # Build for production (8GB memory)
npm run build:analyze  # Build with bundle size analysis
npm run start          # Start production server
npm run start:turbo    # Production server with Turbopack
```

### Quick Development Scripts
```bash
# Use the shorthand script for common commands
./.claude/commands/dev              # Start stable development
./.claude/commands/dev sim          # Simulator-optimized mode
./.claude/commands/dev clean        # Clean cache and start
./.claude/commands/dev help         # Show all available commands

# Advanced utilities
./.claude/commands/dev-utils.sh smart    # Auto-select best mode based on system
./.claude/commands/dev-utils.sh memory   # Check memory usage
./.claude/commands/dev-utils.sh health   # Complete system health check
```

### Testing

```bash
# Unit & Integration Testing
npm run test          # Run Jest tests
npm run test:watch    # Jest in watch mode
npm run test:coverage # Generate coverage report
npm run test:ui       # Jest with verbose output
npm run test:component # Test UI components only
npm run test:domain   # Test domain layer only
npm run test:integration # Integration tests only

# End-to-End Testing
npm run test:e2e      # Run Playwright tests
npm run test:e2e:ui   # Playwright with UI
npm run test:e2e:debug # Debug Playwright tests

# All Tests
npm run test:all      # Run all tests (Jest + Playwright)
```

### Development Tools

```bash
# Cache Management
npm run clean:cache   # Clear Next.js and build cache
npm run clean:all     # Full reset with dependency reinstall

# Analysis
npm run turbo:analyze # Analyze Turbopack performance
npm run lint         # ESLint with TypeScript
```

## Project Structure

```
src/
├── app/                       # Next.js App Router pages
│   ├── (marketing)/          # Marketing landing pages
│   ├── onboarding/           # Onboarding flow pages
│   └── api/                  # API routes
├── modules/                   # Domain-driven feature modules (ALL business logic here)
│   ├── analytics/            # Analytics tracking module
│   ├── marketing/            # Marketing, landing page and lead capture
│   ├── onboarding/           # Onboarding flow logic
│   ├── pricing/              # Pricing plans and AB testing
│   ├── whatsapp-simulator/   # WhatsApp conversation simulator
│   │   ├── domain/           # Entities (Message, Conversation)
│   │   ├── application/      # Use cases & conversation engine
│   │   ├── infra/           # GIF export, factories, services
│   │   ├── ui/              # Simulator components & hooks
│   │   ├── scenarios/       # Predefined conversation flows
│   │   └── __tests__/       # Comprehensive test suite
│   └── shared/               # Shared domain logic
├── components/               # Reusable UI components
│   └── ui/                   # Base UI components (Radix-based)
├── shared/                   # Cross-cutting concerns
│   ├── components/           # Atomic design components
│   ├── hooks/                # Custom React hooks
│   ├── state/                # Redux store configuration
│   └── types/                # TypeScript type definitions
├── store/                    # Redux store exports
├── lib/                      # Utilities and helpers
├── e2e/                      # Playwright end-to-end tests
├── __mocks__/                # Jest mocks and test utilities
└── public/
    ├── gif.worker.js         # Web Worker for GIF processing
    └── sw-simulator.js       # Service Worker for offline support
```

**Nota importante**: Todo el código de negocio está centralizado en `src/modules/`. Las carpetas legacy `domain/`, `presentation/`, `infrastructure/` y `application/` han sido eliminadas y su contenido movido al módulo correspondiente.

## WhatsApp Simulator Module

### Overview

The WhatsApp Simulator is a comprehensive module that simulates realistic WhatsApp conversations with advanced features including GIF export, typing indicators, and multiple conversation scenarios.

### Key Features

- **Realistic Conversation Simulation**: Mimics WhatsApp's UI and behavior patterns
- **Multiple Scenarios**: Pre-built conversation flows (restaurant, medical, loyalty programs)
- **GIF Export**: Export conversations as animated GIFs with Web Worker optimization
- **Typing Indicators**: Realistic typing animation with timing controls
- **Performance Optimized**: Canvas rendering with memory management
- **Accessibility**: Full WCAG 2.1 AA compliance
- **Mobile Responsive**: Optimized for all device sizes

### Architecture Components

#### Domain Layer (`domain/`)

- **Message Entity**: Represents individual WhatsApp messages with metadata
- **Conversation Entity**: Manages message collections and conversation state
- **Events**: Domain events for conversation lifecycle management

#### Application Layer (`application/`)

- **Conversation Engine**: Core business logic for conversation flow
- **Use Cases**: Jump to message, play/pause, reset, speed control
- **Ports**: Interfaces for external dependencies

#### Infrastructure Layer (`infra/`)

- **GIF Export Service**: High-performance GIF generation with optimization
- **Conversation Factory**: Creates conversation instances from scenarios
- **Web Worker Integration**: Background processing for GIF generation

#### UI Layer (`ui/`)

- **WhatsApp Simulator Component**: Main simulator interface
- **Conversation Simulator**: Message rendering and interaction
- **Export Controls**: GIF export interface and settings
- **Custom Hooks**: Reusable logic for conversation flow, timing, GIF export

#### Scenarios (`scenarios/`)

Pre-built conversation templates:

- **Restaurant Reservations**: Booking and confirmation flows
- **Medical Appointments**: Healthcare scheduling scenarios
- **Restaurant Orders**: Food ordering and delivery tracking
- **Loyalty Programs**: Customer reward interactions

### Testing Strategy

Comprehensive test coverage with specialized testing approaches:

- **Domain Tests**: Entity behavior and business rules (95% coverage)
- **Application Tests**: Use case execution and edge cases (90% coverage)
- **UI Tests**: Component rendering and user interactions
- **Integration Tests**: GIF export functionality and performance
- **Accessibility Tests**: WCAG compliance and screen reader support
- **Performance Tests**: Memory usage and rendering optimization

### Usage Examples

```typescript
// Basic conversation simulation
import { WhatsAppSimulator, RestaurantReservationScenario } from '@/modules/whatsapp-simulator';

const scenario = new RestaurantReservationScenario();
<WhatsAppSimulator scenario={scenario} />

// Advanced GIF export
import { useGifExport } from '@/modules/whatsapp-simulator/ui/hooks';

const { exportAsGif, isExporting } = useGifExport({
  optimize: true,
  quality: 0.8,
  width: 320
});
```

## Architecture Patterns

### Domain-Driven Design with Clean Architecture

Each module follows a strict layered architecture:

```
module/
├── domain/           # Business entities and rules
├── application/      # Use cases and ports
├── infra/           # External services and adapters
└── ui/              # React components and hooks
```

### Key Principles Applied

- **Dependency Inversion**: Application defines ports, infrastructure implements adapters
- **Feature Isolation**: Each module is self-contained with minimal cross-dependencies
- **Clean Separation**: Business logic is decoupled from framework specifics
- **Port & Adapters Pattern**: Clear interfaces between layers

## Accessibility Standards

### WCAG 2.1 AA Compliance

The project implements comprehensive accessibility standards to ensure usability for all users:

#### Color Contrast & Visual Design

- **High Contrast Support**: `prefers-contrast: high` media query implementation
- **Minimum Ratios**: 4.5:1 for normal text, 8.2:1 for primary CTAs  
- **Color Independence**: Information conveyed through multiple channels (color + text + icons)
- **Focus Management**: 2-3px visible focus rings with high contrast colors

#### Keyboard Navigation

- **Skip Links**: `Alt + S` shortcut to main content
- **Focus Trapping**: Complete focus management in modal dialogs
- **Logical Tab Order**: Sequential navigation through interactive elements
- **Arrow Navigation**: Enhanced navigation within complex components
- **Escape Handling**: Consistent modal and dropdown dismissal

#### Semantic HTML & ARIA

- **Landmark Roles**: `role="banner"`, `role="navigation"`, `role="main"`
- **Descriptive Labels**: `aria-label` and `aria-describedby` for context
- **Live Regions**: `aria-live` for dynamic content updates
- **State Management**: `aria-expanded`, `aria-selected` for interactive elements
- **Screen Reader Support**: Comprehensive testing with NVDA and JAWS

#### Mobile Accessibility

- **Touch Targets**: Minimum 44px clickable areas
- **Gesture Alternatives**: Keyboard equivalents for all touch interactions
- **Zoom Support**: Content remains functional at 200% zoom
- **Orientation Independence**: Works in portrait and landscape modes

### Accessibility Testing

- **Automated Testing**: Built into Jest test suite and Playwright E2E tests
- **Manual Testing**: Screen reader testing and keyboard-only navigation
- **Audit Reports**: Comprehensive WCAG compliance documentation available

## Current Implementation Status

### Completed Features

- Marketing landing page with sections (Hero, Features, Pricing, Testimonials, FAQ)
- Onboarding flow with multiple steps (Business info, WhatsApp integration, Bot setup, Testing)  
- Analytics module with event tracking and metrics
- Pricing module with AB testing support
- Lead capture forms with validation
- Performance monitoring and Web Vitals tracking
- Service Worker for offline support
- **WhatsApp Simulator**: Full conversation simulation with GIF export capabilities
- **Comprehensive Testing**: Jest + Playwright with 80-95% coverage requirements
- **Accessibility Compliance**: WCAG 2.1 AA standards implementation

### Module Organization

Each module includes:

- README.md documenting the module's purpose
- index.ts for clean exports
- Strict separation between domain, application, and infrastructure layers

## Testing Infrastructure

### Jest Configuration

- **Framework**: Jest with Next.js integration
- **Environment**: jsdom for React component testing
- **Setup**: Custom setup files for mocks and testing utilities
- **Path Mapping**: Full TypeScript path resolution (`@/` prefix)
- **File Mocking**: Static assets and CSS modules mocked
- **Canvas Support**: jest-canvas-mock for GIF generation testing

### Coverage Requirements

```javascript
// Global coverage thresholds
global: {
  branches: 80,
  functions: 80, 
  lines: 90,
  statements: 90
}

// Domain layer (strict requirements)
'src/modules/whatsapp-simulator/domain/': {
  branches: 95,
  functions: 95,
  lines: 95, 
  statements: 95
}

// Application layer
'src/modules/whatsapp-simulator/application/': {
  branches: 90,
  functions: 90,
  lines: 90,
  statements: 90  
}
```

### Playwright E2E Testing

- **Multi-browser Support**: Chrome, Firefox, Safari (desktop + mobile)
- **Test Isolation**: Each test runs in isolated browser context
- **Screenshots**: Automatic failure screenshots and video recording
- **Debugging**: UI mode and debug tools integration
- **CI Integration**: GitHub Actions reporter and optimized settings

### Test Categories

- **Unit Tests**: Individual component and function testing
- **Integration Tests**: Module interaction and data flow
- **Component Tests**: React component rendering and interaction
- **Domain Tests**: Business logic and entity behavior
- **Accessibility Tests**: WCAG compliance and screen reader support
- **Performance Tests**: Memory usage and rendering optimization
- **E2E Tests**: Full user journey testing across browsers

### Testing Commands

```bash
# Coverage focused testing
npm run test:coverage    # Generate detailed HTML coverage report
npm run test:domain      # Test domain layer with strict requirements
npm run test:component   # UI component testing only
npm run test:integration # Cross-module integration tests

# Development testing
npm run test:watch       # Watch mode for active development
npm run test:ui          # Verbose output for debugging
npm run test:all         # Complete test suite (Jest + Playwright)
```

## Development Guidelines

### Styling Strategy - IMPORTANT

**PRIORIDAD DE ESTILOS (en orden estricto):**

1. **Tailwind CSS First**: Usar siempre clases de Tailwind como primera opción
   - Utilizar las utilidades predefinidas de Tailwind
   - Aprovechar las clases de espaciado, colores y tipografía del sistema
   - NO crear clases CSS personalizadas si existe una utilidad de Tailwind

2. **Module CSS para componentes específicos**: Cuando Tailwind no es suficiente
   - Usar archivos `.module.css` para estilos específicos de componentes
   - Mantener los estilos encapsulados y locales al componente
   - Nombrar archivos como `ComponentName.module.css`

3. **Global CSS solo para estilos verdaderamente globales**: Usar con extrema precaución
   - Solo para reset de estilos, variables CSS globales o temas
   - NO añadir estilos de componentes específicos en `globals.css`
   - NO crear archivos CSS adicionales en carpetas de estilos compartidos

**REGLAS ESTRICTAS:**
- ❌ NUNCA crear archivos CSS en `/shared/styles/` para componentes específicos
- ❌ NUNCA añadir estilos de navbar, botones o componentes en `globals.css`
- ❌ NUNCA crear archivos como `navbar-animations.css` o `accessibility.css` separados
- ✅ SIEMPRE usar Tailwind primero
- ✅ SIEMPRE usar `.module.css` si necesitas CSS personalizado
- ✅ SIEMPRE mantener `globals.css` mínimo y limpio

### Component Development

- **UI Library**: Use existing components from `components/ui/` (Radix-based)
- **Atomic Design**: Follow atomic design principles in `shared/components/`
- **Forms**: React Hook Form + Zod validation with comprehensive error handling
- **Accessibility**: WCAG 2.1 AA compliance from the start (focus management, ARIA labels)
- **Animation**: Framer Motion for performant animations and transitions
- **Testing**: Write component tests alongside development (React Testing Library)

### State Management

- **Global State**: Redux Toolkit for complex application state
- **Local State**: React hooks for component-specific state
- **Custom Hooks**: Extract reusable stateful logic into custom hooks
- **Performance**: Use React.memo and useMemo for expensive operations
- **Testing**: Include state transitions in unit tests

### API Integration

- **API Routes**: Next.js API routes in `app/api/`
- **Service Layer**: Service classes in module's `infra/services/`
- **Repository Pattern**: Data access abstraction for testability
- **Error Handling**: Comprehensive error boundaries and user feedback
- **Analytics**: Track API performance and user interactions

### Testing Requirements

- **Coverage**: Maintain minimum thresholds (Domain: 95%, Application: 90%)
- **Test Types**: Unit, integration, component, and accessibility tests
- **TDD Approach**: Write tests before implementation for critical business logic
- **Mocking**: Mock external dependencies and APIs for reliable testing
- **E2E Testing**: Use Playwright for critical user journeys

### Code Quality Standards

- **TypeScript**: Strict mode enabled with comprehensive type definitions
- **Linting**: ESLint with Next.js and TypeScript rules
- **Clean Architecture**: Maintain clear separation of concerns between layers
- **Performance**: Monitor bundle size and runtime performance metrics

### Performance Optimization

#### Core Web Vitals & Monitoring

- **Real-time Tracking**: Web Vitals 5.1 with @vercel/speed-insights integration
- **Performance Metrics**: LCP, FID, CLS monitoring and reporting
- **Error Tracking**: Sentry integration with replay functionality
- **Analytics Integration**: Vercel Analytics and PostHog for user behavior

#### Memory & Resource Management

- **Development Optimization**: Multiple NODE_OPTIONS configurations for memory limits
- **GIF Processing**: Web Workers for background GIF generation to prevent UI blocking
- **Canvas Optimization**: Memory-efficient rendering with cleanup procedures
- **Service Worker**: Offline support and asset caching (sw-simulator.js)

#### Build & Runtime Optimizations

- **Turbopack Integration**: Fast development with Webpack fallback
- **Bundle Analysis**: Built-in tools for bundle size monitoring
- **Image Optimization**: Next.js Image component with optimization utilities
- **Progressive Enhancement**: Service Worker for offline functionality

#### Development Performance

```bash
# Memory-optimized development commands
npm run dev:stable    # 8GB memory + debugging
npm run dev:memory    # 12GB memory + enhanced GC
npm run dev:debug     # Performance debugging with telemetry
npm run clean:cache   # Clear build caches for fresh start
```

#### Performance Testing

- **Lighthouse Integration**: Automated performance auditing in CI/CD
- **Memory Profiling**: Built-in memory usage monitoring for GIF export
- **Load Testing**: Component performance under heavy usage scenarios
- **Mobile Optimization**: Performance testing on low-end devices

## Known Issues & Solutions

### Development Environment Issues

#### jsx-dev-runtime Bug with Turbopack

- **Issue**: Random jsx-dev-runtime errors during development
- **Solutions**: Multiple dev commands available for different scenarios

  ```bash
  npm run dev:stable    # Enhanced memory + debugging
  npm run dev:webpack   # Webpack fallback when Turbopack fails
  npm run dev:debug     # Verbose logging for troubleshooting
  npm run dev:clean     # Clean cache + restart
  ```

#### Memory Issues with Large Projects

- **Issue**: Node.js memory exhaustion during development
- **Solution**: Memory-optimized development commands

  ```bash
  npm run dev:memory    # 12GB memory + enhanced GC
  npm run dev:stable    # 8GB memory (recommended)
  ```

#### Windows-Specific Issues

- **Issue**: Different environment variable syntax on Windows
- **Solution**: Windows-specific commands available

  ```bash
  npm run dev:windows        # Windows environment variables
  npm run dev:clean:windows  # Windows cache cleanup
  ```

### Production Considerations

#### GIF Export Performance

- **Issue**: Large GIF files can impact performance
- **Solution**: Web Worker implementation with optimization options
- **Monitoring**: Built-in memory profiling and cleanup procedures

#### Test Coverage Maintenance

- **Issue**: Strict coverage requirements (95% for domain layer)
- **Solution**: Automated coverage reporting with detailed breakdown
- **Commands**: `npm run test:coverage` for analysis

### Build & Deployment

#### Bundle Size Monitoring

- **Issue**: Large bundle sizes with media processing libraries
- **Solution**: Built-in bundle analysis tools
- **Command**: `npm run turbo:analyze` for performance insights

## Next Steps for Development

### Priority 1: Backend Integration

1. Complete Supabase lead capture integration
2. Implement WhatsApp Cloud API webhook handling
3. Add comprehensive error tracking with Sentry

### Priority 2: Enhanced Features  

4. User authentication and session management
5. Advanced analytics dashboard with PostHog
6. Real-time conversation analytics

### Priority 3: Production Readiness

7. Performance optimization for mobile devices
8. Advanced accessibility testing automation
9. Comprehensive E2E test coverage for critical paths
10. CI/CD pipeline with automated testing and deployment

### Future Enhancements

- Multi-language support for international markets
- Advanced GIF export options (quality, formats, optimization)
- Integration with additional messaging platforms
- Advanced conversation flow builder

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fjpedrosa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
