## assistant-api-test

> This document outlines the architectural patterns and conventions that should be followed when developing code for this project.

# Company Architecture Guidelines

This document outlines the architectural patterns and conventions that should be followed when developing code for this project.

## Backend Architecture - Layered Approach

### Layer Structure

**API Layer**
- HTTP interface, request validation, response formatting, auth middleware
- Only depends on Application layer

**Application Layer**
- Use case orchestration, transaction management, business workflows
- Defines interfaces for Infrastructure layer

**Domain Layer**
- Pure business logic, entities, validation rules
- Zero external dependencies

**Infrastructure Layer**
- Database, external APIs, file system
- Implements Application interfaces

### Backend Rules
- Domain has no dependencies
- Application defines interfaces, Infrastructure implements
- API only handles HTTP
- No circular dependencies

## Frontend Architecture - Three Layer Approach

### Layer Structure

**Controllers**
- API calls, data fetching, global state management, heavy lifting
- One per route/major feature

**Composites**
- Intermediate components combining multiple components
- Local state, user interactions
- Examples: Card, Form, SearchBar

**Components**
- Basic UI elements with internal state only
- No data fetching
- Examples: Button, Input, Modal

### Frontend Rules
- Controllers handle external data
- Composites combine components
- Components stay pure UI with internal state only
- Data flows down, events up

## Development Flow

### Backend Development
1. Start with Domain layer
2. Define Application interfaces
3. Implement Infrastructure
4. Build API last

### Frontend Development
1. Build Components first
2. Compose into Composites
3. Create Controllers for data management

## Implementation Guidelines

When working on this codebase:
- Follow the layered architecture strictly
- Respect dependency rules
- Keep components pure and focused on their layer responsibilities
- Use the prescribed development flow for new features

---
> Source: [extrainjulian/assistant-api-test](https://github.com/extrainjulian/assistant-api-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
