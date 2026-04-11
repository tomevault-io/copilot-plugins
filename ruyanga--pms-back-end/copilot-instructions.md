## pms-back-end

> This AGENTS.md file provides comprehensive guidance for AI assistants like GitHub Copilot when working with the Kangalos Project Management Platform backend codebase.

# AGENTS.md Guide for Kangalos Backend

This AGENTS.md file provides comprehensive guidance for AI assistants like GitHub Copilot when working with the Kangalos Project Management Platform backend codebase.

## Pr### Implementation Guidelines

- Use a reusable pagination service or utility
- Combine with Prisma's `skip` and `take` for efficient querying
- Include sort and filter parameters in pagination links
- Always validate and sanitize pagination parameters Overview

Kangalos is a project management platform built for the University of Rwanda and its partner institutions. It serves as a central hub for tracking research projects, student theses, community innovations, and externally funded initiatives.

## Technology Stack

- **Framework**: NestJS (v11)
- **Language**: TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Email**: Nodemailer with Handlebars templates
- **Validation**: class-validator and class-transformer
- **Logging**: Winston with daily-rotate-file
- **Documentation**: Swagger/OpenAPI

## Project Structure for AI Assistant Navigation

- `/src`: Source code
  - `/generated`: Auto-generated Prisma client code (don't modify directly)
  - `/logger`: Custom logging implementation
  - `/mail`: Email service implementation with templates
  - `/prisma`: Prisma service for database access
- `/prisma`: Prisma schema and migration files
- `/scripts`: Utility scripts including entrypoint for Docker
- `/logs`: Application logs (not tracked in git)

## Database Schema

The project uses Prisma ORM with PostgreSQL. The schema is defined in `/prisma/schema.prisma`. Key models include:

- `OrganisationUnit`: Represents the hierarchical structure of university departments

## Coding Conventions for AI Assistance

### General Conventions

- Use TypeScript for all new code
- Follow NestJS architectural patterns (controllers, services, modules, DTOs)
- Maintain strict typing throughout the codebase
- Use dependency injection following the NestJS pattern
- Add meaningful comments for complex logic **Comments & JSDoc:** Document all public methods with parameter and return JSDoc comments.
- Follow existing code style and formatting
- **README:** Maintain a project-level `README.md` with setup, environment variables, and contributing guidelines.
- **Environment Variables:** Store secrets in `.env`; include a `.env.example` with placeholders.

### NestJS Module Guidelines

- Always use NestJS CLI generate commands to generate specific components:
  - `nest g module [name]` - Generate a new module
  - `nest g controller [name]` - Generate a new controller
  - `nest g service [name]` - Generate a new service
  - `nest g class [name]/dto/[name].dto` - Generate a new DTO class
  - `nest g interface [name]/interfaces/[name].interface` - Generate a new interface
  - `nest g pipe [name]` - Generate a new pipe
  - `nest g filter [name]` - Generate a new filter
  - `nest g guard [name]` - Generate a new guard
  - `nest g decorator [name]` - Generate a new custom decorator
- Keep modules focused on specific domain functionality
- Use proper dependency injection
- Register providers in appropriate modules
- Use Global modules sparingly and with purpose

### Code Organization Standards

- DTOs should be in a `dto` folder within each module and they should have swagger notations from "@nestjs/swagger" and validates
- Services should handle business logic
- Controllers should be thin and delegate to services with proper swagger notations from "@nestjs/swagger"

### Well-Implemented Patterns to Follow

The **organisation module** (`/src/organisation/`) serves as the gold standard for implementation patterns. When creating new modules or DTOs, reference these files:

**DTO Structure Examples:**

- `src/organisation/dto/create-organisation-unit.dto.ts` - Proper create DTO with validation and Swagger documentation
- `src/organisation/dto/query-organisation-unit.dto.ts` - Complete query DTO with pagination, sorting, and filtering
- `src/organisation/dto/update-organisation-unit.dto.ts` - Clean update DTO using PartialType

**Controller Pattern:**

- `src/organisation/organisation.controller.ts` - Comprehensive controller with proper Swagger documentation, error handling, and endpoint organization

**Service Implementation:**

- `src/organisation/organisation.service.ts` - Well-structured service with proper error handling, logging, and business logic

**Reusable Infrastructure:**

- `src/infrastructure/infrastructure.service.ts` - Provides reusable utilities for common operations like checking record existence and preventing duplicates

### DTO Best Practices (Based on Organisation Module)

1. **Create DTOs:**
   - Use `@ApiProperty` with descriptions and examples
   - Apply proper validation decorators (`@IsString`, `@IsNotEmpty`, `@IsUUID`, etc.)
   - Include optional fields with `@IsOptional` and `required: false` in ApiProperty

2. **Query DTOs:**
   - Always include pagination parameters (page, limit)
   - Add sorting parameters (sortBy, sortOrder)
   - Include search functionality with `@IsOptional` and `@IsString`
   - Use `@Type(() => Number)` for numeric transformations
   - Set sensible defaults for pagination and sorting
   - Use `@IsIn()` for enum-like validations

3. **Update DTOs:**
   - Use `PartialType` from `@nestjs/swagger` to make all fields optional
   - Keep it simple and extend the create DTO

## Command Line Instructions

Always run these commands to check code quality before submitting changes:

```bash
# Fix linting issues automatically
npm run lint:fix

# Check for linting errors
npm run lint

# Type check the TypeScript code
npm run type-check

# Full check (type check + lint)
npm run check-all

# Build the project to check if the task you executed is working
npm run build

# Prisma commands
# Generate Prisma client
npm run prisma:generate

# Create a migration
npm run prisma:migrate:dev

# Apply migrations in production
npm run prisma:migrate:deploy

# Open Prisma Studio for DB visualization
npm run prisma:studio
```

## Development Workflow

1. Make sure to run `npm run prisma:generate` after any schema changes
2. Always validate changes with `npm run check-all` before committing
3. When making changes to the database schema, create proper migrations
4. Use the logger service for all logging, avoid console.log
5. Add appropriate validation to DTOs using class-validator

## Error Handling

- Use NestJS exception filters for consistent error responses
- Properly catch and handle errors in async operations
- Use appropriate HTTP status codes in responses
- Log errors with sufficient context for debugging

## Email Templates

Email templates are located in `/src/mail/templates` and use Handlebars syntax. When modifying or creating templates:

1. Maintain consistent styling with existing templates
2. Test templates with sample data before deployment
3. Include proper variables in the mail service

## Environment Configuration

The application uses environment variables for configuration. Required variables include:

- `DATABASE_URL`: PostgreSQL connection string
- `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_USER`, `EMAIL_PASS`: SMTP settings
- `EMAIL_SECURE`: Whether to use secure connection ("true" or "false")
- `EMAIL_FROM_NAME`: Sender name for emails

## Testing Guidelines

When implementing or modifying tests:

1. no tests are used in our repository please delete any \*.spec.ts files that are generated by nest commands

## Pull Request Guidelines

When helping create a PR, ensure it:

0. When you can not create a PR for your changes please create a readme file and follow .github/pull_request_template.md and put it in prs folder
1. Includes a clear description of the changes
2. References any related issues
3. Ensures all tests pass and new code is covered by tests
4. Keeps changes focused on a single concern
5. Passes all linting and type checking

## Api structure

1. Pagination & Filtering: All GET collection endpoints must support pagination (page, limit), sorting, filtering, and optional full-text search via query parameters.
2. **HTTP Status Codes:** Return appropriate status codes (`200`, `201`, `204`, `400`, `401`, `403`, `404`, `500`). please use exceptions as nestjs provides them

### API Response Format

All API responses should follow this standardized structure:

```json
{
  "status": true | false,
  "message": "Success message or error description",
  "data": {
    // The actual response data (array or object)
  },
  "meta": {
    // Metadata including pagination information when applicable
  }
}
```

### Example MD json Data Generation

When creating new API endpoints, **always generate 20 example JSON objects** in one single markdown file for easy copy-paste usage. These examples must:

1. **Match the DTO structure** - Follow the exact field names, types, and validation rules from the created DTOs
2. **Align with the Prisma model** - Respect database constraints, relationships, and field requirements
3. **Reflect project context** - Use realistic data that fits the Kangalos project (University of Rwanda, research projects, theses, etc.)

**Example JSON Requirements:**

- Create examples for both request payloads (matching Create DTOs) and response data (matching the model)
- Include edge cases and validation examples
- Use contextually appropriate data (university departments, Rwandan names, academic fields, etc.)
- Place examples in markdown files with clear headings and copy-paste friendly formatting
- Include both valid and invalid examples to demonstrate validation

**File Structure for Examples:**

```text
/drafts/
  endpoint-name-examples.md
    - Valid Request Examples (13 examples)
    - Invalid Request Examples (7 examples)
```

### Pagination Implementation

For paginated endpoints, implement the following:

1. **Query Parameters:**
   - `page`: Current page number (default: 1)
   - `limit`: Items per page (default: 10)
   - `sortBy`: Field to sort by
   - `sortDir`: Sort direction ('asc' or 'desc')
   - `search`: Optional search term for filtering

2. **Response Format for Paginated Data:**

```json
{
  "status": true,
  "message": "Successfully retrieved items",
  "data": [
    // Array of items
    { "id": 1, ... },
    { "id": 2, ... },
    // ...
  ],
  "meta": {
    "pagination": {
      "total": 100,        // Total number of items
      "count": 10,         // Number of items in current page
      "perPage": 10,       // Items per page
      "currentPage": 1,    // Current page number
      "totalPages": 10,    // Total number of pages
      "links": {
        "first": "/api/resource?page=1&limit=10",
        "last": "/api/resource?page=10&limit=10",
        "prev": null,      // null when on first page
        "next": "/api/resource?page=2&limit=10"  // null when on last page
      }
    }
  }
}
```

**Implementation Guidelines:**

<!-- - Use a reusable pagination service or utility -->

- Combine with Prisma's `skip` and `take` for efficient querying
- Include sort and filter parameters in pagination links
- Always validate and sanitize pagination parameters

### Error Response Format

For error responses, follow this structure:

```json
{
  "status": false,
  "message": "Error description",
  "data": null,
  "meta": {
    "error": {
      "code": "ERROR_CODE",
      "details": {
        // Optional additional error details
      }
    }
  }
}
```

### Common Response Types

**Success Response (Single Item):**

```json
{
  "status": true,
  "message": "Item retrieved successfully",
  "data": {
    "id": 1,
    "name": "Example Item",
    "createdAt": "2025-07-09T10:30:00Z",
    ...
  },
  "meta": {}
}
```

**Success Response (Create/Update):**

```json
{
  "status": true,
  "message": "Item created successfully",
  "data": {
    "id": 1,
    "name": "New Item",
    ...
  },
  "meta": {}
}
```

**Success Response (Delete):**

```json
{
  "status": true,
  "message": "Item deleted successfully",
  "data": null,
  "meta": {}
}
```

**Validation Error:**

```json
{
  "status": false,
  "message": "Validation failed",
  "data": null,
  "meta": {
    "error": {
      "code": "VALIDATION_ERROR",
      "details": {
        "name": ["Name is required"],
        "email": ["Invalid email format"]
      }
    }
  }
}
```

## Windows Command Line Reference

When AI needs to interact with files and folders on Windows (using PowerShell or CMD):

### File Operations

```powershell
# View file content
Get-Content path\to\file.txt        # PowerShell
type path\to\file.txt               # CMD

# Create empty file
New-Item -Path path\to\file.txt -ItemType File    # PowerShell
type nul > path\to\file.txt                       # CMD

# Copy file
Copy-Item source.txt destination.txt              # PowerShell
copy source.txt destination.txt                   # CMD

# Delete file
Remove-Item path\to\file.txt                      # PowerShell
del path\to\file.txt                              # CMD

# Move/rename file
Move-Item old.txt new.txt                         # PowerShell
move old.txt new.txt                              # CMD
ren old.txt new.txt                               # CMD (rename)
```

### Directory Operations

```powershell
# List directory contents
Get-ChildItem                       # PowerShell
dir                                 # CMD

# Create directory
New-Item -Path path\to\dir -ItemType Directory    # PowerShell
mkdir path\to\dir                                 # CMD

# Remove directory
Remove-Item -Path path\to\dir -Recurse            # PowerShell (with contents)
rmdir /s /q path\to\dir                           # CMD (with contents)

# Navigate directories
Set-Location path\to\dir                          # PowerShell
cd path\to\dir                                    # CMD

# Current directory path
Get-Location                                      # PowerShell
cd                                                # CMD
```

### File Content Manipulation

```powershell
# Find in files
Select-String -Path "*.ts" -Pattern "searchterm"  # PowerShell
findstr /s /i "searchterm" *.ts                   # CMD

# Replace in file (PowerShell)
(Get-Content file.txt) -replace 'old', 'new' | Set-Content file.txt
```

### Common npm Commands

```powershell
# Install package
npm install package-name

# Install dev dependency
npm install --save-dev package-name

# Run script from package.json
npm run script-name
```

This AGENTS.md file is intended to help AI assistants understand the project structure and follow best practices when generating code or providing guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RUYANGA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RUYANGA)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
