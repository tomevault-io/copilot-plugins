## logisti-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Node.js delivery app that connects Merchants with Truck Owners for international shipments. The application uses Express.js with MongoDB and includes role-based authentication, real-time tracking, document management, and WhatsApp notifications.

## Development Commands

### Core Commands

- `npm run dev` - Start development server with nodemon
- `npm start` - Start production server
- `npm run dev:all` - Run both backend and React frontend concurrently
- `npm run format` - Format code with Prettier

### Client Commands

- `npm run client` - Start React frontend development server
- `npm run client:build` - Build React frontend for production
- `npm run client:install` - Install frontend dependencies

### Database Commands

- `npm run db:create-indexes` - Create optimized MongoDB indexes
- `npm run db:monitor-indexes` - Monitor index performance
- `npm run db:optimize` - Run both index creation and monitoring

### Admin Management

- `npm run create-admin` - Create admin user interactively
- `npm run logs` - Run log management utilities
- `npm run logs:list` - List available log files
- `npm run logs:clean` - Clean old log files

### Testing

- Run tests with: `npx jest` or `./tests/run-tests.sh`
- Test environment uses MongoDB Memory Server
- Jest configuration in `jest.config.js`
- Test files in `tests/` directory

## Architecture Overview

### Directory Structure

```
src/
├── controllers/     # Route handlers organized by domain
│   ├── admin/      # Admin-specific controllers
│   ├── auth/       # Authentication controllers
│   ├── truck/      # Fleet management controllers
│   └── ...
├── middleware/     # Express middleware functions
├── models/         # Mongoose models (User, Truck, Shipment, Application, Document)
├── routes/         # Route definitions
├── services/       # Business logic layer
├── utils/          # Utility functions and helpers
├── config/         # Configuration files
└── scripts/        # Maintenance and setup scripts
```

### Key Models

- **User**: Role-based (Admin, Merchant, Truck Owner, Driver)
- **Truck**: Fleet management with driver assignments
- **Shipment**: Core shipping entity with status workflow
- **Application**: Bidding system for shipments
- **Document**: File management with entity relationships

### Authentication & Authorization

- JWT-based authentication
- Role-based access control middleware in `src/middleware/authMiddleware.js`
- User roles: Admin, Merchant, Truck Owner, Driver
- Admin user auto-created on first startup

### Database

- MongoDB with Mongoose ODM
- Optimized indexes for query performance
- Connection configuration in `src/config/database.js`
- Database monitoring utilities in `src/utils/dbMonitor.js`

### API Documentation

- Swagger UI available at `/api-docs`
- OpenAPI specification at `/api-docs-json/json`
- Comprehensive documentation for all endpoints with examples

### Monitoring & Health Checks

- Health check endpoints at `/health/*`
- Prometheus metrics at `/metrics`
- System monitoring with CPU, memory, and disk metrics
- Database health monitoring
- External API monitoring support

### Real-time Features

- Socket.io integration for live updates
- Location tracking for shipments
- Real-time notifications

### File Storage

- On-premises document storage
- Upload middleware with validation
- Document linking with shipments and applications

### Notification System

- WhatsApp integration via Twilio
- Event-based notification triggers
- Configurable notification preferences

## Development Guidelines

### Code Conventions

- Follow existing patterns in controllers and services
- Use async/await for asynchronous operations
- Implement proper error handling with try/catch blocks
- Use middleware for cross-cutting concerns
- Follow RESTful API design principles

### Security

- All sensitive endpoints require authentication
- Role-based access control enforced
- CSRF protection enabled
- Rate limiting implemented
- Security headers configured

### Testing Strategy

- Unit tests for business logic in services
- Integration tests for API endpoints
- Use MongoDB Memory Server for test database
- Test fixtures and utilities in `tests/utils/`

### Error Handling

- Centralized error handling in `src/middleware/errorHandler.js`
- Structured logging with Winston
- Error tracking and metrics

## Important Implementation Notes

- Admin user is automatically created on first startup
- Database indexes should be created for optimal performance
- The application supports Docker deployment with docker-compose
- Environment variables configuration required (see `env.sample`)
- Client-side React application is in `/client` directory
- Comprehensive logging system with daily rotation
- Rate limiting configured for different endpoint types

## Active Technologies
- TypeScript 5 on Next.js 14.2.35 (App Router) with React 18 + @tanstack/react-query 5.90, zustand 5.0, socket.io-client 4.8, react-leaflet 4.2, react-hook-form 7.71, zod 4.3, Radix UI, Tailwind CSS 3.4, cmdk 1.1, lucide-react 0.563, sonner 2.0, date-fns 4.1 (007-production-frontend)
- N/A (frontend consumes backend REST API + Socket.io) (007-production-frontend)

## Recent Changes
- 007-production-frontend: Added TypeScript 5 on Next.js 14.2.35 (App Router) with React 18 + @tanstack/react-query 5.90, zustand 5.0, socket.io-client 4.8, react-leaflet 4.2, react-hook-form 7.71, zod 4.3, Radix UI, Tailwind CSS 3.4, cmdk 1.1, lucide-react 0.563, sonner 2.0, date-fns 4.1

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OmarEhab007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OmarEhab007)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
