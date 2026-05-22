## roadmap-workout-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a NestJS-based workout tracker API project from roadmap.sh. The application will implement user authentication with JWT, workout management, exercise data seeding, and progress tracking with reporting features.

### Key Features to Implement
- User authentication and authorization with JWT
- Exercise database with seeding functionality
- Workout plan creation, updating, and scheduling
- Progress tracking and report generation
- RESTful API endpoints with OpenAPI documentation

## Development Commands

### Core Development
- `npm run start:dev` - Start development server with hot reload
- `npm run start:debug` - Start with debug mode enabled
- `npm run build` - Build the production application
- `npm run start:prod` - Run production build

### Code Quality
- `npm run lint` - Run ESLint with auto-fix
- `npm run format` - Format code with Prettier
- `npm run test` - Run unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:cov` - Run tests with coverage report
- `npm run test:e2e` - Run end-to-end tests

### Single Test Execution
- `npm run test -- --testNamePattern="specific test name"` - Run specific unit test
- `npm run test:e2e -- --testNamePattern="specific e2e test"` - Run specific e2e test

## Architecture

### Technology Stack
- **Framework**: NestJS with Express
- **Language**: TypeScript with strict configuration
- **Testing**: Jest for unit tests, Supertest for e2e tests
- **Code Quality**: ESLint + Prettier with TypeScript ESLint rules

### Project Structure
- `src/` - Main application source code
  - `main.ts` - Application entry point (runs on port 3000)
  - `app.module.ts` - Root application module
  - `app.controller.ts` - Basic controller with health check endpoint
  - `app.service.ts` - Application service layer
- `test/` - End-to-end test files
- Configuration files use modern standards (ESM, flat config for ESLint)

### Development Patterns
- Uses NestJS decorators and dependency injection
- Module-based architecture with clear separation of concerns
- TypeScript with decorators enabled for NestJS functionality
- Jest configuration supports both unit tests (in `src/`) and e2e tests (in `test/`)

### Database Requirements
Based on the project requirements, you'll need to implement:
- Relational database integration (PostgreSQL, MySQL, or SQLite)
- User entity with authentication fields
- Exercise entity with categories/muscle groups
- Workout entity with scheduling capabilities
- Many-to-many relationships between workouts and exercises
- Data seeding for exercise database

### Security Considerations
- JWT authentication implementation required
- User authorization for workout access
- Input validation and sanitization
- Secure password handling (hashing)

### API Design
- RESTful endpoints following NestJS conventions
- OpenAPI/Swagger documentation integration
- Proper HTTP status codes and error handling
- Request/response DTOs with validation

---
> Source: [chinsanchung/roadmap-workout_tracker](https://github.com/chinsanchung/roadmap-workout_tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
