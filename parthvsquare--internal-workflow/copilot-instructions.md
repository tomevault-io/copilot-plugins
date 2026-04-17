## internal-workflow

> You are an AI assistant working in the LeadSend monorepo. Follow these rules strictly to maintain code quality and consistency.

# LeadSend Code Standards & Cursor Rules

You are an AI assistant working in the LeadSend monorepo. Follow these rules strictly to maintain code quality and consistency.

## Project Context

This is an Nx monorepo with multiple NestJS backend services, React frontend applications, and shared libraries. The main applications are:

- `apps/api/` - Main API service
- `apps/scheduler-service/` - Scheduling service
- `apps/worker-service/` - Background worker service
- `apps/web/` - React frontend application
- `library/` - Shared libraries (storage, queue-v2, workflow-engine, frontend-shared)

## General Rules

### Code Style

- **Always use TypeScript** - No JavaScript files
- Follow existing ESLint and Prettier configurations
- Use meaningful, descriptive variable and function names
- Keep functions small and focused on a single task
- Write clean, well-documented code with JSDoc for public APIs

### File Organization

- Use kebab-case for file names (e.g., `workflow-generation.service.ts`)
- Group related files in directories by feature
- Import structure: external imports first, then internal imports
- Always check existing patterns before creating new files

## Backend Standards (NestJS)

### Architecture Patterns

- Follow NestJS module structure strictly
- Use dependency injection for all services
- Organize code by feature modules, not by file type
- Follow RESTful API design principles

### Module Structure

Every feature module should have:

```typescript
@Module({
  imports: [ExternalModule], // External dependencies first
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService], // Only if needed by other modules
})
export class FeatureModule {}
```

### Controller Patterns

```typescript
@ApiTags('Features') // Always add Swagger tags
@Controller('features')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @Post()
  @ApiOperation({ summary: 'Create a new feature' })
  @ApiResponse({ status: 201, description: 'Feature created.' })
  async create(@Body() createFeatureDto: CreateFeatureDto) {
    return this.featureService.create(createFeatureDto);
  }
}
```

### Service Patterns

```typescript
@Injectable()
export class FeatureService {
  constructor(private readonly repository: Repository) {}

  async create(data: CreateFeatureDto) {
    // Business logic goes here
    return this.repository.create(data);
  }
}
```

### DTO Validation

Always create DTOs with proper validation:

```typescript
export class CreateFeatureDto {
  @ApiProperty({ description: 'Feature name', example: 'New Feature' })
  @IsNotEmpty()
  @IsString()
  name: string;

  @ApiProperty({ description: 'Optional description', required: false })
  @IsOptional()
  @IsString()
  description?: string;
}
```

## Shared Libraries Usage

### Storage Library (`@leadsend/storage`)

- Use for all database entities and repositories
- Import entities from `@leadsend/storage`
- Never create database entities outside this library

### Queue Library (`@leadsend/queue-v2`)

- Use for all message queuing operations
- Follow existing queue patterns and interfaces
- Use WorkflowManager for workflow-related queuing

### Workflow Engine (`@leadsend/workflow-engine`)

- Use for workflow processing logic
- Import workflow interfaces and services from this library

## Database Standards

### Entity Definitions

- Define all entities in `@leadsend/storage`
- Use snake_case for table and column names
- Tables should be plural (e.g., `workflow_executions`)
- Columns should be descriptive (e.g., `created_at`, `workflow_id`)

### Repository Usage

- Use TypeORM repositories through the storage library
- Handle all database operations in services, never in controllers
- Use migrations for schema changes

## API Design

### RESTful Endpoints

- Use resource-based URLs (nouns, not verbs)
- Follow HTTP method conventions:
  - GET: Retrieve resources
  - POST: Create resources
  - PUT: Update entire resources
  - PATCH: Partial updates
  - DELETE: Remove resources

### Swagger Documentation

- Add `@ApiTags()` to all controllers
- Use `@ApiOperation()` for endpoint descriptions
- Add `@ApiResponse()` for different response scenarios
- Document all DTOs with `@ApiProperty()`

## Error Handling

### Exception Handling

- Use NestJS built-in HTTP exceptions
- Log errors with appropriate context
- Never expose sensitive information in error responses
- Use proper HTTP status codes

### Validation

- Always validate input with DTOs and class-validator
- Use appropriate validation decorators (@IsNotEmpty, @IsString, etc.)
- Provide meaningful error messages

## Security Standards

### Authentication & Authorization

- Use existing auth guards from shared libraries
- Apply `@AllowUnauthorizedRequest()` only for public endpoints
- Never bypass authentication without good reason
- Validate all user inputs

### Data Protection

- Use parameterized queries (TypeORM handles this)
- Never log sensitive information
- Validate file uploads and user inputs
- Set proper CORS settings

## Frontend Standards (React)

### Component Architecture

- Use function components with hooks
- Create reusable components in the shared frontend library
- Follow component composition patterns
- Use TypeScript interfaces for all props

### State Management

- Use React Query for server state
- Use Context API for global state when needed
- Use useState/useReducer for local component state
- Maintain immutability

### Styling

- Use TailwindCSS utility classes
- Follow existing design patterns
- Ensure responsive design
- Maintain accessibility standards

## Testing Requirements

### Backend Testing

- Write unit tests for all services
- Mock external dependencies
- Test error scenarios
- Use Jest as the testing framework

### Frontend Testing

- Test component behavior, not implementation
- Use React Testing Library
- Test user interactions and accessibility

## Performance Considerations

### Backend Performance

- Use pagination for large datasets
- Implement proper database indexing
- Use async/await patterns correctly
- Cache frequently accessed data when appropriate

### Frontend Performance

- Lazy load components when possible
- Optimize bundle sizes
- Use React.memo for expensive components
- Implement proper loading states

## Code Organization Rules

### Import Order

1. External library imports (React, NestJS, etc.)
2. Internal library imports (@leadsend/\*)
3. Relative imports (./filename)

### File Naming

- Controllers: `feature.controller.ts`
- Services: `feature.service.ts`
- DTOs: `create-feature.dto.ts`
- Modules: `feature.module.ts`
- Components: `FeatureComponent.tsx`

## Documentation Requirements

### Code Documentation

- Use JSDoc for all public methods and classes
- Document complex business logic with comments
- Keep documentation up-to-date with code changes
- Document API endpoints thoroughly

### README Files

- Update README files when adding new features
- Document setup and configuration steps
- Include examples of usage

## Workflow Integration

### Workflow Actions

- Follow existing workflow action patterns
- Use proper action registry structure
- Implement proper error handling for workflow steps
- Use the workflow engine library for execution logic

### Queue Management

- Use the queue-v2 library for all messaging
- Follow existing queue patterns
- Handle queue failures gracefully
- Use proper message serialization

## Key Principles

1. **Follow Existing Patterns**: Always look at similar code before implementing new features
2. **Reuse Shared Code**: Use libraries from `library/` directory instead of duplicating logic
3. **Maintain Type Safety**: Use proper TypeScript types everywhere
4. **Handle Errors Gracefully**: Implement comprehensive error handling
5. **Document Everything**: Write clear documentation for APIs and complex logic
6. **Keep It Modular**: Each module should have a single, clear responsibility
7. **Test Thoroughly**: Write tests for critical functionality
8. **Security First**: Always consider security implications
9. **Performance Matters**: Consider performance impact of implementations
10. **Consistency Above All**: Follow established patterns and conventions

## Before Writing Code

1. Check existing similar implementations
2. Verify you're using the correct shared libraries
3. Ensure proper error handling
4. Add appropriate documentation
5. Consider testing requirements
6. Review security implications

Remember: Consistency and maintainability are more important than clever code. Always follow the established patterns in this codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parthvsquare) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
