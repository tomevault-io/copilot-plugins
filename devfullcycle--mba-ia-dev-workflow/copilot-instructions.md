## typeorm

> Use when writing TypeORM entities, database configuration, or data source setup. Covers entity patterns with dual identifiers, soft delete, and Data Mapper.


# TypeORM Pattern

## Rules

1. **Dual identifier**: every entity has an auto-incremented `id` (integer, primary key, internal only) and a `uuid` (v4, unique, exposed to the external world). Never expose `id` in API responses — always use `uuid`
2. **Table naming**: lowercase, plural, snake_case (e.g., `users`, `course_prerequisites`, `quiz_attempts`)
3. **Synchronize**: `synchronize: true` in development, `synchronize: false` in production
4. **Soft delete only**: use `deletedAt` column with TypeORM's `@DeleteDateColumn()` — never hard delete (RN-022)
5. **Data Mapper pattern**: entities are plain data holders, repositories handle persistence — no Active Record

## Entity example

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
} from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @PrimaryGeneratedColumn('uuid')
  uuid: string;

  @Column({ length: 100 })
  name: string;

  @Column({ unique: true })
  email: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date | null;
}
```

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
