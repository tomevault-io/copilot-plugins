## sentinelmusic

> - All NestJS modules, services, controllers, and providers **must** be defined using TypeScript `@Module()`, `@Injectable()`, and `@Controller()` decorators, following official NestJS conventions.


## NestJS API Rules

- All NestJS modules, services, controllers, and providers **must** be defined using TypeScript `@Module()`, `@Injectable()`, and `@Controller()` decorators, following official NestJS conventions.
- Each NestJS service **must** use explicit dependency injection in the constructor and avoid using the `any` type.
- Circular dependencies **must** be identified and refactored using forward references or architectural changes.
- All public endpoints in controllers **must** define DTO classes for request validation, and DTOs **must** use `class-validator` decorators for every property.
- Exception handling **must** be centralized using global filters, and custom exceptions should extend `HttpException`.
- The `ConfigService` **must** be used for environment variables, never process.env directly outside configuration files.
- Asynchronous code **must** use `async/await` syntax and return Promises where appropriate, avoiding `.then/.catch` chains.
- Unit tests **must** be written for services and controllers using Jest, and coverage **must** be reviewed before merging to main.
- Sensitive data and secrets **must never** be logged or exposed through exceptions or responses.
- All routes **must** be versioned and RESTful; query and path parameters **must** be type-validated.

## General Code Quality Rules

- **Type Safety**: Always use TypeScript strict mode. Avoid `any` type without strong justification.
- **Modularity**: Keep files small and focused. Use hooks and composable patterns. Avoid huge files.
- **Testing**: Write tests for all business logic. Maintain minimum coverage thresholds (Services: 80%, Controllers: 70%).
- **Security**: Never expose secrets, validate all inputs, sanitize data, use parameterized queries.
- **Performance**: Optimize for performance. Use code splitting, lazy loading, and efficient database queries.
- **Documentation**: Document complex logic and public APIs. Keep comments up to date.

## Code Style Rules

- **No Console Logs**: NEVER use `console.log` in production code. Use proper logging services.
- **Strict Mode**: Always use TypeScript strict mode and enable all strict checks.
- **Naming**: Use PascalCase for classes/components, camelCase for variables/functions, UPPER_SNAKE_CASE for constants.
- **File Organization**: One class/component per file. Use barrel exports for packages.
- **Error Handling**: Always implement proper error handling. Never expose sensitive data in error messages.

## What NOT to Do

- NEVER commit secrets or API keys
- NEVER use `any` type without justification
- NEVER ignore TypeScript errors
- NEVER skip tests for new features
- NEVER use `process.env` directly (use ConfigService)
- NEVER write raw SQL queries (use Prisma)
- NEVER ignore linting errors
- NEVER commit code that doesn't pass type checking
- NEVER use `console.log` in production code
- NEVER expose sensitive data in error messages or responses

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openann19)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/openann19)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
