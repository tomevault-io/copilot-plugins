## aiassessmenttool

> description: Apply repository pattern and consistent data access standards when implementing database operations to ensure maintainability, type safety, and proper error handling

___
description: Apply repository pattern and consistent data access standards when implementing database operations to ensure maintainability, type safety, and proper error handling
globs: "src/**/*.{ts,tsx}"
___

# Database Access Patterns

## Context

Database access should follow consistent patterns to ensure maintainable, secure, and efficient code. The repository pattern provides a standardized approach to data access that abstracts the underlying database implementation and enforces business rules consistently.

## Requirements

### Repository Pattern Implementation

- All database operations MUST be performed through repository classes, never directly with Prisma client in API routes or services
- Repositories MUST follow a consistent naming convention: `{Entity}Repository`
- Repository methods MUST include tenant isolation in all queries
- Repositories MUST use standardized error handling
- **Repositories SHOULD implement versioning** for audit-critical entities (see @081-data-versioning-standards.mdc)
- Repository methods MUST have consistent naming:
  - `findById`: Find a single entity by ID
  - `findByX`: Find entities by a specific attribute
  - `findAll`: Find all entities (with optional filtering)
  - `create`: Create a new entity
  - `update`: Update an existing entity
  - `delete`: Delete an entity
  - `count`: Count entities

### Multi-Tenant Data Access

- All database queries MUST include tenant context (`organizationId`)
- Cross-tenant operations MUST be explicitly authorized and audited
- Repository methods MUST validate tenant context for all operations

### Error Handling

- Repositories MUST catch all database errors and transform them to application-specific errors
- Database errors MUST NOT leak to API clients
- Error handling MUST be consistent across repositories
- Error messages MUST be user-friendly while preserving technical details for logging

### Transaction Management

- Complex operations affecting multiple entities MUST use transactions
- Transactions MUST be handled at the repository level
- Transaction boundaries MUST be clearly defined
- Transaction retry logic SHOULD be implemented for transient errors

### Query Performance

- Repositories SHOULD optimize database queries for performance
- N+1 query problems MUST be avoided by using proper eager loading
- Large result sets MUST be paginated

## Examples

<example>
// UserRepository with proper implementation
import { PrismaClient, User, Prisma } from '@prisma/client';
import { DatabaseError, NotFoundError } from '../errors/DatabaseError';

export class UserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string, organizationId: string): Promise<User | null> {
    try {
      return await this.prisma.user.findFirst({
        where: {
          id,
          organizationId // Multi-tenancy enforcement
        }
      });
    } catch (error) {
      throw new DatabaseError('Failed to find user by ID', error);
    }
  }

  async findAll(
    organizationId: string,
    options?: {
      skip?: number;
      take?: number;
      orderBy?: Prisma.UserOrderByWithRelationInput;
    }
  ): Promise<User[]> {
    try {
      return await this.prisma.user.findMany({
        where: { organizationId },
        skip: options?.skip,
        take: options?.take,
        orderBy: options?.orderBy
      });
    } catch (error) {
      throw new DatabaseError('Failed to find users', error);
    }
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    try {
      return await this.prisma.user.create({ data });
    } catch (error) {
      throw new DatabaseError('Failed to create user', error);
    }
  }

  async update(
    id: string,
    organizationId: string,
    data: Prisma.UserUpdateInput
  ): Promise<User> {
    try {
      return await this.prisma.user.update({
        where: {
          id,
          organizationId // Multi-tenancy enforcement
        },
        data
      });
    } catch (error) {
      throw new DatabaseError('Failed to update user', error);
    }
  }

  async createWithProfile(
    userData: Prisma.UserCreateInput,
    profileData: Prisma.ProfileCreateInput
  ): Promise<User> {
    try {
      // Use transaction to ensure atomic operation
      return await this.prisma.$transaction(async (tx) => {
        const user = await tx.user.create({
          data: userData
        });

        await tx.profile.create({
          data: {
            ...profileData,
            userId: user.id,
            organizationId: user.organizationId
          }
        });

        return user;
      });
    } catch (error) {
      throw new DatabaseError('Failed to create user with profile', error);
    }
  }
}
</example>

<example>
// API endpoint using repository pattern
import { NextApiRequest, NextApiResponse } from 'next';
import { UserRepository } from '../../../repositories/UserRepository';
import { withAuth } from '../../../middleware/auth';

export default withAuth(async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { organizationId } = req.auth;
  
  // Initialize repository
  const userRepository = new UserRepository();
  
  // GET request to fetch users
  if (req.method === 'GET') {
    try {
      // Parse pagination parameters
      const page = parseInt(req.query.page as string) || 1;
      const limit = Math.min(parseInt(req.query.limit as string) || 20, 100);
      const offset = (page - 1) * limit;
      
      // Fetch users with pagination
      const users = await userRepository.findAll(
        organizationId,
        { skip: offset, take: limit }
      );
      
      // Return paginated response
      return res.status(200).json({
        data: users,
        pagination: {
          page,
          limit,
          total: await userRepository.count(organizationId)
        }
      });
    } catch (error) {
      console.error('Error fetching users:', error);
      return res.status(500).json({ error: 'Failed to fetch users' });
    }
  }
  
  // Method not allowed
  return res.status(405).json({ error: `Method ${req.method} not allowed` });
});
</example>

<example>
// Repository factory for dependency injection
import { PrismaClient } from '@prisma/client';
import { UserRepository } from './UserRepository';
import { ProjectRepository } from './ProjectRepository';
import { TaskRepository } from './TaskRepository';

// Singleton Prisma instance
const prisma = new PrismaClient();

// Repository factory
export const repositories = {
  user: new UserRepository(prisma),
  project: new ProjectRepository(prisma),
  task: new TaskRepository(prisma)
};

// For testing, we can create repositories with a test client
export function createTestRepositories(testPrisma: PrismaClient) {
  return {
    user: new UserRepository(testPrisma),
    project: new ProjectRepository(testPrisma),
    task: new TaskRepository(testPrisma)
  };
}
</example>

<example type="invalid">
// ❌ AVOID: Direct database access in API route
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default async function handler(req, res) {
  if (req.method === 'GET') {
    // Direct database access without repository pattern
    const users = await prisma.user.findMany();
    return res.status(200).json(users);
  }
}
</example>

<example type="invalid">
// ❌ AVOID: Missing tenant isolation
async function getProjects() {
  const prisma = new PrismaClient();
  
  // Missing tenant isolation
  return await prisma.project.findMany();
}
</example>

<example type="invalid">
// ❌ AVOID: Inconsistent error handling
export class InconsistentRepository {
  constructor(private prisma: PrismaClient) {}
  
  // Inconsistent error handling - sometimes throwing errors
  async findById(id: string) {
    return await this.prisma.user.findUnique({ where: { id } });
  }
  
  // Sometimes returning null
  async findByEmail(email: string) {
    try {
      return await this.prisma.user.findUnique({ where: { email } });
    } catch (error) {
      return null;
    }
  }
  
  // Sometimes logging and re-throwing
  async create(data) {
    try {
      return await this.prisma.user.create({ data });
    } catch (error) {
      console.error('Error creating user:', error);
      throw error;
    }
  }
}
</example>

## Measuring Compliance

- Codebase scan to identify direct Prisma client usage outside repositories
- Review of repository implementations for consistent patterns
- Validation of tenant isolation in all database queries
- Testing of error handling for various database failure scenarios

## See Also

### Documentation
- **`.cursor/docs/ai-workflows.md#schema-first-development`** - **CRITICAL:** Schema inspection patterns!
- **`.cursor/rules/003-cursor-system-overview.mdc`** - System overview (READ THIS FIRST)

### Tools (USE THESE!)
- **`.cursor/tools/inspect-model.sh`** - **CRITICAL:** Check field names, types, relationships!
  ```bash
  ./.cursor/tools/inspect-model.sh YourModel
  # Shows: fields, types, organizationId pattern, relationships
  ```

### Comprehensive Guides
- **`guides/ORM-Usage-Guide.md`** - **GOLD STANDARD:** Repository pattern, query patterns!
- **`guides/Database-Query-Optimization-Guide.md`** - **CRITICAL:** Query optimization patterns!
- **`guides/Database-Schema-Guide.md`** - Schema design, relationships
- **`guides/Database-Resilience-Guide.md`** - Resilient access patterns

### Related Rules
- **@002-rule-application.mdc** - **CRITICAL:** Source of Truth Hierarchy (schema first!)
- **@025-multi-tenancy.mdc** - **CRITICAL:** Organization scoping in ALL queries!
- @081-data-versioning-standards.mdc - Data versioning and audit trails (implement in repositories)
- @061-database-integration.mdc - Database integration
- @082-database-performance-budgets.mdc - Performance optimization
- @067-database-security.mdc - Secure access patterns
- @376-database-test-isolation.mdc - Testing access patterns

### Quick Start
1. **ALWAYS:** `.cursor/tools/inspect-model.sh YourModel` (check organizationId!)
2. **Repository Pattern:** Use patterns from `guides/ORM-Usage-Guide.md`
3. **Multi-Tenancy:** EVERY query must scope by organizationId (@025-multi-tenancy.mdc)
4. **Optimize:** Follow `guides/Database-Query-Optimization-Guide.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spar65) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
