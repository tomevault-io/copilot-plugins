## united-tactical-defense

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Frontend Development
```bash
# Start frontend development server
cd frontend && npm start

# Build frontend for production
cd frontend && npm run build:prod

# Build frontend with optimized assets
cd frontend && npm run build:optimized

# Run frontend tests
cd frontend && npm test

# Analyze bundle size
cd frontend && npm run analyze

# Optimize assets before build
cd frontend && npm run optimize-assets

# Serve production build locally
cd frontend && npm run serve:prod

# Preload critical assets
cd frontend && npm run preload-critical-assets
```

### Full Stack Development
```bash
# Install all dependencies (root + frontend)
npm run install-all

# Start both frontend and backend concurrently
npm start

# Build frontend only
npm run build
```

### Testing
```bash
# Run all tests
npm run test:all

# Run backend API tests
npm run test:api

# Run service layer tests  
npm run test:services

# Run integration tests
npm run test:integration

# Run E2E tests
npm run test:e2e

# Run frontend tests
npm run test:frontend

# Run performance tests
npm run test:performance

# Generate test coverage report
npm run test:coverage

# Generate comprehensive test report
npm run test:report
```

### Database Operations
```bash
# Run database migrations up
npm run migrate

# Run database migrations down
npm run migrate:down

# Test database connection pooling
npm run test:db-pooling
```

## Project Architecture

### Frontend Architecture (React + TypeScript)
- **Framework**: React 18 with TypeScript
- **Routing**: React Router DOM with lazy loading  
- **Styling**: SCSS with component-scoped styles + Styled Components for dynamic styles
- **State Management**: React hooks with context for global state
- **Build Tool**: Create React App with custom optimizations
- **Analytics**: Google Analytics 4 integration with custom tracking
- **Icons**: FontAwesome React components and React Icons
- **Charts**: Chart.js with react-chartjs-2 for dashboard analytics

### Backend Architecture (Hybrid: Deprecated + Serverless)
- **Current**: Serverless functions (Netlify) with mock data fallbacks in `frontend/netlify/functions/`
- **Deprecated**: Express.js server with SQLite database (in `/deprecated/`)
- **External APIs**: LeadConnector API for appointment booking
- **Database**: Transitioned from SQLite to external services
- **Development Mode**: Mock implementations with `logMockOperation` indicators in console

### Key Directories
- `frontend/src/components/`: React components organized by feature
- `frontend/src/services/`: API layer and external service integrations
- `frontend/src/utils/`: Utility functions and helper modules
- `deprecated/`: Legacy Express.js backend (reference only)
- `netlify/functions/`: Serverless API endpoints

## Code Architecture Patterns

### Component Organization
- **Landing Page**: Modular sections (Hero, Programs, Testimonials, etc.) in `frontend/src/components/landing/`
- **Dashboard**: Analytics dashboard with charts and metrics in `frontend/src/components/dashboard/`
- **Booking**: Appointment booking flow with calendar integration in `frontend/src/components/booking/`
- **Common**: Shared components (Footer, modals, etc.) in `frontend/src/components/common/`
- **Form Components**: Booking forms and modals in `frontend/src/components/Form/`

### API Layer Pattern
- Mock implementations for development (`frontend/src/services/api.ts`)
- Fallback to real endpoints when available
- Comprehensive TypeScript interfaces for all data models
- Error handling with graceful degradation

### Analytics Integration
- Custom analytics service with GA4
- Page visit tracking with user journey analysis
- Conversion tracking and funnel analysis
- Real-time metrics dashboard

## External Service Integrations

### LeadConnector API
- **Purpose**: External appointment booking system
- **Endpoint**: `https://backend.leadconnectorhq.com/appengine/appointment`
- **Format**: Multipart form data submission
- **Fields**: `first_name`, `last_name`, `email`, `phone`, `selected_slot`, `tag`

### Firebase Services
- **Analytics**: Event tracking and user behavior analysis
- **Configuration**: Environment-based initialization
- **Hosting**: Static asset delivery (if used)

### Netlify Platform
- **Hosting**: Frontend deployment with automatic builds
- **Functions**: Serverless API endpoints
- **Forms**: Contact form handling
- **Redirects**: SPA routing support

## Development Workflow

### Local Development Setup
1. Install dependencies: `npm run install-all`
2. Start development: `npm start` (runs both frontend and deprecated backend concurrently)
3. Frontend runs on `http://localhost:3000`
4. Deprecated backend API runs on `http://localhost:3001`
5. For frontend-only development: `cd frontend && npm start`

### Testing Strategy
- **Unit Tests**: Component and utility testing with Jest/React Testing Library
- **Integration Tests**: API endpoint testing with Mocha/Chai
- **Performance Tests**: Load testing with autocannon
- **E2E Tests**: Full user journey testing

### Build & Deployment
- **Development**: `npm start` for hot reload
- **Production**: `npm run build` creates optimized build  
- **Optimized Production**: `cd frontend && npm run build:optimized` (includes asset optimization)
- **Deployment**: Netlify automatic deployment from main branch using `build:optimized`
- **Asset Optimization**: Automatic image format conversion, video compression, bundle optimization
- **Local Testing**: `cd frontend && npm run serve:prod` to test production build locally

## TypeScript Configuration

### Frontend (React)
- **Target**: ES5 with modern library support
- **Module System**: ESNext with Node resolution
- **Strict Mode**: Enabled for type safety
- **Base URL**: `src` for absolute imports
- **JSX**: React JSX transform

### Type Definitions
- Comprehensive interfaces for all API responses
- Form data types with validation schemas
- Analytics event types
- Component prop interfaces

## Special Considerations

### Calendar Integration
- Custom calendar component with booking restrictions
- Date validation preventing past bookings
- Month navigation with business logic constraints
- Integration with external booking API

### Asset Optimization
- Automatic image format conversion (WebP, AVIF)
- Video compression and multiple format support
- Bundle splitting and lazy loading
- Critical asset preloading

### Analytics & Tracking
- Privacy-compliant user tracking
- Custom event definitions
- Conversion funnel analysis
- Geographic and device analytics

### Performance Optimizations
- Component lazy loading with Suspense
- Route-based code splitting
- Asset compression and caching
- CDN integration for static assets

## Environment Variables

### Frontend
```bash
REACT_APP_GA4_MEASUREMENT_ID=your_ga4_id
REACT_APP_GA4_CONVERSION_ID=your_conversion_id
REACT_APP_API_BASE_URL=/api
```

### Backend (Deprecated)
```bash
NODE_ENV=development
PORT=3001
DB_PATH=./unitedDT.db
EXTERNAL_APPOINTMENT_API=https://backend.leadconnectorhq.com/appengine/appointment
```

## Troubleshooting

### Common Issues
- **Build Failures**: Check TypeScript errors and missing dependencies
- **API Errors**: Verify external service endpoints and authentication
- **Asset Loading**: Ensure proper path resolution for optimized assets
- **Analytics**: Confirm GA4 configuration and event tracking

### Debug Commands
- `npm run test:report` - Generate comprehensive test report
- `cd frontend && npm run analyze` - Analyze frontend bundle size with source-map-explorer
- `npm run test:performance` - Load test API endpoints with autocannon
- `cd frontend && npm run build:analyze` - Generate bundle analysis with webpack-bundle-analyzer
- Check browser console for mock API indicators during development

### Asset Optimization Commands
- `cd frontend && npm run optimize-assets` - Optimize images and videos before build
- `cd frontend && npm run preload-critical-assets` - Generate critical asset preloading
- `cd frontend && npm run build:optimized` - Build with full asset optimization pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orcaclubpro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
