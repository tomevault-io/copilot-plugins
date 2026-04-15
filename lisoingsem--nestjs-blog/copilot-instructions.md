## nestjs-blog

> This is a **NestJS Blog API** with modular architecture featuring:

# Cursor Rules for NestJS Blog Project

## 🏗️ Project Overview

This is a **NestJS Blog API** with modular architecture featuring:
- **GraphQL API** with code-first approach
- **JWT Authentication** with role-based access control
- **Prisma ORM** with PostgreSQL database
- **Modular structure** with clear separation of concerns
- **CLI system** with nest-commander for database management
- **Security-first** approach with comprehensive guards and validation

## 📚 Comprehensive Rules Documentation

For complete rules and patterns, refer to these organized files:

### 🏗️ **Architecture & Structure**
- **Architecture**: `.cursor/rules/architecture.mdc` - Module boundaries, structure, dependencies, patterns
- **GraphQL**: `.cursor/rules/graphql.mdc` - Code-first approach, resolvers, types, schema patterns

### 🗄️ **Data & Services**
- **Database**: `.cursor/rules/database.mdc` - Prisma schemas, migrations, service patterns
- **Services**: `.cursor/rules/services.mdc` - Service layer patterns, business logic, error handling

### 🔧 **Development & Operations**
- **CLI**: `.cursor/rules/cli.mdc` - Command patterns and database management
- **Security**: `.cursor/rules/security.mdc` - Authentication, authorization, security patterns
- **Testing**: `.cursor/rules/testing.mdc` - Unit tests, integration tests, testing patterns
- **Deployment**: `.cursor/rules/deployment.mdc` - Docker, environment config, deployment scripts

## Key Principles

### 1. 🏗️ Architecture Rules
- **Core modules** (`src/modules/`) contain business-critical functionality
- **Feature modules** (`src/modules/`) contain domain-specific features  
- **Common utilities** (`src/common/`) contain shared infrastructure
- **No circular dependencies** - Dependencies flow downward only
- **Dynamic loading** - Modules auto-discovered, no manual imports in app.module.ts

### 2. 🗄️ Database Rules
- **Single schema** - Use standard `prisma/schema.prisma` file
- **Standard Prisma workflow** - Use standard Prisma commands directly
- **Service-only pattern** - Services use Prisma directly, no repository abstraction
- **Migrations** - Use descriptive names, always test

### 3. 📁 File Naming Conventions
- **Services**: `*.service.ts`
- **Modules**: `*.module.ts`
- **DTOs**: `*.input.ts`, `*.args.ts`
- **Commands**: `*.command.ts`
- **Guards**: `*.guard.ts`
- **Interceptors**: `*.interceptor.ts`

### 4. 📦 Import Rules
```typescript
// ✅ Use path aliases
import { UserService } from '@modules/users';
import { PrismaService } from '@common/prisma';

// ❌ Avoid relative paths (except in CLI)
import { UserService } from '../../modules/users/user.service';

// Import order: Node modules → Internal modules → Types
```

### 5. 🔧 Module Structure (REQUIRED)
Each module MUST have:
```
module-name/
├── *.service.ts      # Business logic + data access
├── *.module.ts       # NestJS module
├── dto/              # Input/output types
├── index.ts          # Public API exports
```

### 6. 🛡️ Security Patterns
```typescript
// Authentication & Authorization
@UseGuards(JwtGuard, PermissionGuard)
@RequirePermissions('user:create')
async createUser() {}

// Error handling
throw new NotFoundException(`User with ID ${id} not found`);
throw new ConflictException('Email already exists');

// Audit logging
await this.auditService.logUserAction(userId, 'CREATE', 'user', newUser.id);
```

### 7. 🗄️ Database Patterns
```typescript
// Service-only pattern (recommended)
@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}
  
  async findById(id: number): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }
  
  async create(data: CreateUserInput): Promise<User> {
    // Business logic + data access in one place
    const existing = await this.prisma.user.findUnique({
      where: { email: data.email }
    });
    if (existing) throw new ConflictException('Email exists');
    
    return this.prisma.user.create({ data });
  }
}

// Interface pattern (for external services only)
export interface IEmailService {
  sendWelcome(user: User): Promise<void>;
}

@Injectable()
export class EmailService implements IEmailService {
  async sendWelcome(user: User): Promise<void> {
    // Implementation
  }
}
```

### 8. 🔧 CLI Commands
```bash
# Database management
yarn run cli db                    # Generate Prisma client
yarn run cli db generate           # Generate client only
yarn run cli db migrate [name]     # Create/apply migrations
yarn run cli db studio             # Open Prisma Studio
yarn run cli db status             # Show migration status
yarn run cli db health             # Database health check
```

### 9. 📊 Logging Standards
```typescript
// Structured logging with emojis
this.logger.log('✅ Operation completed successfully');
this.logger.warn('⚠️ Warning message');
this.logger.error('❌ Error occurred');

// Include context
this.logger.log(`User ${userId} created successfully`);
```

### 10. 🧪 Testing Patterns
```typescript
describe('UserService', () => {
  let service: UserService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: PrismaService,
          useValue: {
            user: {
              findUnique: jest.fn(),
              create: jest.fn(),
            },
          },
        },
      ],
    }).compile();
    
    service = module.get<UserService>(UserService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  it('should create user successfully', async () => {
    // Arrange, Act, Assert pattern
  });
});
```

## Current Project State

**Existing modules**:
- `auth/` - JWT authentication
- `users/` - User management
- `permissions/` - Role-based access control  
- `audit/` - Activity tracking

**Database schema**: Single `prisma/schema.prisma` file

**CLI system**: nest-commander based with database management commands

**Dependencies**: NestJS + Prisma + GraphQL + JWT + bcrypt + nest-commander

## AI Assistant Guidelines

When generating code:
1. **Follow modular architecture** - Respect modules/common boundaries
2. **Use service-only pattern** - Services use Prisma directly, no repository abstraction
3. **Include proper validation** - DTOs, guards, exception handling
4. **Consider security** - Authentication, authorization, audit logging
5. **Follow naming conventions** - File names, class names, database tables
6. **Use the CLI system** - For database operations
7. **Maintain type safety** - Interfaces for external services only
8. **Include tests** - Unit tests for services, mock PrismaService
9. **Document appropriately** - JSDoc for APIs, clear variable names
10. **Performance considerations** - Pagination, indexing, query optimization 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lisoingsem) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
