## carpoollive

> This is a comprehensive carpooling and bus booking platform built with React.js (frontend), Node.js/Express (backend), and React Native (mobile).

# BlaBlaCar Platform - Development Rules

## Project Overview

This is a comprehensive carpooling and bus booking platform built with React.js (frontend), Node.js/Express (backend), and React Native (mobile).

## Tech Stack

- **Frontend**: React.js 18 + TypeScript + Material-UI + Redux Toolkit
- **Backend**: Node.js + Express + TypeScript + PostgreSQL + Prisma ORM
- **Mobile**: React Native + TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Real-time**: Socket.io for chat and notifications

## Code Style & Standards

### TypeScript

- Use strict TypeScript with proper type definitions
- Prefer interfaces over types for object shapes
- Use enums for constants and status values
- Always define return types for functions
- Use proper generic types where applicable

### React Components

- Use functional components with hooks
- Prefer named exports over default exports
- Use proper prop interfaces for all components
- Implement proper error boundaries
- Use React.memo for performance optimization when needed

### File Naming & Structure

- Use PascalCase for component files (e.g., `UserProfile.tsx`)
- Use camelCase for utility files (e.g., `apiClient.ts`)
- Use kebab-case for route files (e.g., `user-profile.ts`)
- Group related files in feature-based directories

### API Design

- Use RESTful conventions for API endpoints
- Implement proper error handling and status codes
- Use JWT for authentication
- Implement rate limiting and security measures
- Use proper HTTP methods (GET, POST, PUT, DELETE, PATCH)

### Database

- Use Prisma ORM for all database operations
- Follow naming conventions for models and fields
- Implement proper relationships and constraints
- Use migrations for schema changes
- Implement proper indexing for performance

## Development Guidelines

### State Management

- Use Redux Toolkit for global state
- Use React Query for server state
- Use local state for component-specific data
- Implement proper loading and error states

### Error Handling

- Implement comprehensive error boundaries
- Use proper error logging and monitoring
- Provide user-friendly error messages
- Handle network errors gracefully

### Performance

- Implement code splitting and lazy loading
- Use React.memo and useMemo appropriately
- Optimize bundle size and loading times
- Implement proper caching strategies

### Security

- Validate all user inputs
- Implement proper authentication and authorization
- Use HTTPS in production
- Sanitize data before database operations
- Implement rate limiting and DDoS protection

## Project-Specific Rules

### BlaBlaCar Features

- **User Management**: Implement proper user registration, authentication, and profile management
- **Carpooling**: Handle ride creation, booking, and management
- **Bus Booking**: Integrate with multiple bus operators
- **Payment**: Implement secure payment processing
- **Chat**: Real-time communication between users
- **Safety**: Implement verification and safety features

### Code Organization

- Keep components small and focused
- Use proper separation of concerns
- Implement proper testing for critical features
- Use consistent naming conventions across the project

### Dependencies

- Keep dependencies up to date
- Use specific versions for critical packages
- Avoid unnecessary dependencies
- Document why each dependency is needed

## Testing

- Write unit tests for utility functions
- Write integration tests for API endpoints
- Write component tests for React components
- Maintain good test coverage
- Use proper mocking and test data

## Documentation

- Document complex business logic
- Keep README files updated
- Document API endpoints
- Maintain proper code comments
- Update architecture documentation

## Git & Deployment

- Use meaningful commit messages
- Follow conventional commit format
- Implement proper CI/CD pipelines
- Use feature branches for development
- Maintain clean git history

## Remember

- This is a production application - prioritize security and reliability
- Follow accessibility best practices
- Implement proper logging and monitoring
- Consider scalability and performance from the start
- Maintain code quality and consistency across the project
  description:
  globs:
  alwaysApply: false

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WebxInfotech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WebxInfotech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
