## rzume-web

> Rzume web application project context and architecture


# Rzume Web - Project Context

## Project Overview
**Type**: Job application tracking and resume management platform  
**Framework**: Angular 18.2.0  
**UI Library**: Angular Material  
**State Management**: Service-based with RxJS BehaviorSubjects  
**Authentication**: Google OAuth + JWT tokens  
**Backend API**: REST API at localhost:7103 (development)

## Technology Stack

### Core Dependencies
- **Angular**: 18.2.0 (core, material, cdk)
- **TypeScript**: 5.5+
- **RxJS**: 7.8.0
- **Angular Material**: 18.2.0
- **Google OAuth**: @abacritt/angularx-social-login
- **JWT**: @auth0/angular-jwt

### Development Tools
- **Testing**: Jasmine, Karma, Cypress, Playwright
- **Build**: Angular CLI
- **Styling**: SCSS with custom theme system
- **Deployment**: Docker + Google Cloud Run

## Application Architecture

### Routing Structure
```
/ → /auth (default redirect)
/auth
  ├── /login
  ├── /register
  ├── /onboard
  ├── /reset-password
  ├── /request-pass-reset
  └── /email-confirmation

/main (protected by AuthGuardService)
  ├── /dashboard
  └── /profile-management
```

### Core Services
- **AuthenticationService**: User authentication and session management
- **ApiService**: HTTP client wrapper with interceptors
- **StorageService**: Session/local storage abstraction
- **GoogleAuthService**: Google OAuth integration
- **JobApplicationService**: Job tracking CRUD operations
- **JobApplicationStateService**: Centralized job state management
- **SearchStateService**: Shared search/filter state
- **ScreenManagerService**: Responsive breakpoint detection
- **LoaderService**: Global loading state
- **AnalyticsService**: Mixpanel + Google Tag Manager integration

### State Management Pattern
- Service-based state with BehaviorSubject
- Observables with `shareReplay` for performance
- Components subscribe via `takeUntil` pattern
- No NgRx or other state libraries

### Authentication Flow
1. User logs in via Google OAuth or email/password
2. Backend returns JWT access + refresh tokens
3. Tokens stored in sessionStorage
4. AuthInterceptor adds token to all API requests
5. AuthGuardService protects /main routes

## Styling System

### SCSS Architecture
- **Theme Variables**: `src/app/styles/variables.scss`
- **Typography**: `src/app/styles/fonts.scss`
- **Global Styles**: `src/styles.scss`
- **Component Styles**: Scoped SCSS files

### Responsive Breakpoints
- **Mobile**: Default (0-598px)
- **Tablet**: 599px and up
- **Desktop**: 950px and up

### Typography Scale
- Font weights: 300, 400, 500, 600, 700, 800
- Font sizes: xs (10px) to 6xl (48px)
- Mobile-first responsive scaling

### Color Palette
- Primary: Green theme (#4CAF50 variants)
- Background: White/light gray
- Text: Dark gray (#333)
- Error: Red (#f44336)
- Success: Green (#4CAF50)

## Component Patterns

### Page Components (Containers)
- Located in `src/app/pages/`
- Handle routing and layout
- Coordinate multiple child components
- Manage page-level state

### Presentation Components
- Located in `src/app/components/`
- Reusable across features
- Input/Output driven
- Minimal business logic

### Form Components
- Configuration-based inputs
- Implement ControlValueAccessor
- Floating label design
- Built-in validation states

## API Integration

### HTTP Interceptors
- **AuthInterceptor**: Adds JWT token to requests
- **AnalyticsInterceptor**: Tracks API calls

### Error Handling
- Global error handler service
- User-friendly error messages
- Automatic retry for failed requests

## Testing Strategy

### Unit Tests
- Jasmine + Karma
- Test all components and services
- Mock external dependencies
- Coverage target: 80%+

### E2E Tests
- Cypress for user flows
- Playwright for cross-browser testing
- Test critical paths (auth, dashboard, job CRUD)

## Build & Deployment

### Development
```bash
npm start  # Runs on localhost:4200
```

### Production Build
```bash
npm run build:prod  # Outputs to dist/
```

### Docker Deployment
- Multi-stage build (Node.js + Nginx)
- Environment config via config.json
- Deployed to Google Cloud Run
- Port 8080 for Cloud Run compatibility

## Feature Flags
- Managed via ConfigService
- Loaded from config.json at runtime
- Controls feature visibility

## Analytics
- Mixpanel for user behavior tracking
- Google Tag Manager for marketing
- Custom event tracking throughout app

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Avwerosuoghene) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
