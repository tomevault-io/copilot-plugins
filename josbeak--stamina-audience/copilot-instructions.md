## stamina-audience

> This document provides a concise summary of the code standards and design patterns to follow in the LeadSend monorepo. For detailed guidelines, refer to `BACKEND_STANDARDS.md` and `FRONTEND_STANDARDS.md`.

 # LeadSend Code Standards & Design Patterns

This document provides a concise summary of the code standards and design patterns to follow in the LeadSend monorepo. For detailed guidelines, refer to `BACKEND_STANDARDS.md` and `FRONTEND_STANDARDS.md`.

## General Standards

### Coding Style

- Use TypeScript for all code
- Follow ESLint and Prettier configurations
- Use meaningful variable and function names
- Keep functions small and focused on a single task
- Write clean, well-documented code

### File Organization

- Use kebab-case for file names (e.g., `feature-component.tsx`)
- Group related files in directories
- Keep import structure consistent (external imports first, then internal)

### Documentation

- Document complex logic with comments
- Use JSDoc for public APIs
- Keep documentation up-to-date with code changes

## Backend Standards

### Architecture

- Follow NestJS module structure
- Use dependency injection for services
- Organize by feature modules
- Follow RESTful API design principles

### Code Organization

- Controllers handle HTTP requests
- Services contain business logic
- DTOs validate input/output
- Entities define database schema

### Data Access

- Use TypeORM repositories
- Handle database queries in services
- Use migrations for schema changes
- Follow consistent naming conventions for database objects

### Examples

**Module Definition**:
```typescript
@Module({
  imports: [DependencyModule],
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService],
})
export class FeatureModule {}
```

**Controller Definition**:
```typescript
@Controller('features')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @Get()
  findAll() {
    return this.featureService.findAll();
  }
}
```

**Service Definition**:
```typescript
@Injectable()
export class FeatureService {
  constructor(private readonly repository: Repository) {}

  findAll() {
    return this.repository.find();
  }
}
```

## Frontend Standards

### Architecture

- Use function components with hooks
- Organize by pages and components
- Use shared library for common functionality
- Follow component composition patterns

### State Management

- Use React Query for server state
- Use Context API for global state
- Use useState/useReducer for local state
- Follow immutability principles

### Component Design

- Keep components focused and reusable
- Use TypeScript interfaces for props
- Follow naming conventions
- Use proper component composition

### Styling

- Use TailwindCSS for styling
- Follow design system guidelines
- Ensure responsive and accessible design
- Use utility-first approach

### Examples

**Component Definition**:
```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  onClick,
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

**Hook Usage**:
```typescript
export function useFetchData(id: string) {
  return useQuery({
    queryKey: ['data', id],
    queryFn: () => fetchData(id),
    enabled: !!id,
  });
}
```

## Design Patterns to Follow

### Backend Patterns

- **Repository Pattern**: Abstract database operations
- **Dependency Injection**: Use NestJS DI system
- **DTO Pattern**: Validate and transfer data
- **Guard Pattern**: Protect routes
- **Module Pattern**: Organize code by feature

### Frontend Patterns

- **Custom Hook Pattern**: Extract and reuse logic
- **Context Provider Pattern**: Share state
- **Compound Component Pattern**: Create related components
- **Render Props Pattern**: Share rendering logic
- **Container/Presentational Pattern**: Separate concerns

### Integration Patterns

- **Event-Driven Architecture**: Use message queues
- **CQRS**: Separate read and write operations
- **Circuit Breaker**: Handle service failures
- **API Gateway**: Centralize API access
- **Service Discovery**: Locate services dynamically

## Key Rules to Remember

1. **Follow Existing Patterns**: Look at similar code before implementing new features
2. **Reuse Shared Code**: Use libraries from the `library/` directory
3. **Maintain Type Safety**: Use proper TypeScript types
4. **Test Your Code**: Write unit and integration tests
5. **Handle Errors**: Implement proper error handling
6. **Document APIs**: Use Swagger for backend, JSDoc for frontend
7. **Keep It Modular**: Each module should have a single responsibility
8. **Maintain Consistency**: Follow established patterns and conventions
9. **Consider Performance**: Optimize for performance when necessary
10. **Security First**: Follow security best practices

By following these guidelines, we ensure that the codebase remains maintainable, scalable, and consistent.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JOSBEAK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
