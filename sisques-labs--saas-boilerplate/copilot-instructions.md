## saas-boilerplate

> Always implement the infrastructure layer following Clean Architecture principles with repositories, mappers, entities, and DTOs. The infrastructure layer provides concrete implementations of domain interfaces.


# Infrastructure Layer Architecture Rule

Always implement the infrastructure layer following Clean Architecture principles with repositories, mappers, entities, and DTOs. The infrastructure layer provides concrete implementations of domain interfaces.

## Infrastructure Layer Structure

The infrastructure layer must be organized in the following structure:

```
infrastructure/
├── database/              # Database implementations
│   ├── typeorm/          # TypeORM write repositories
│   │   ├── entities/     # TypeORM entities
│   │   ├── mappers/      # TypeORM mappers
│   │   └── repositories/ # TypeORM repository implementations
│   └── mongodb/          # MongoDB read repositories
│       ├── dtos/         # MongoDB DTOs
│       ├── mappers/       # MongoDB mappers
│       └── repositories/ # MongoDB repository implementations
└── {other-services}/      # Other infrastructure services (storage, external APIs, etc.)
```

## TypeORM Write Repositories

### Structure

TypeORM repositories implement write repository interfaces and handle persistence of aggregates to PostgreSQL. They must:

- Extend `BaseTypeormMasterRepository` or `BaseTypeormTenantRepository`
- Implement the domain write repository interface
- Use mappers to convert between entities and aggregates
- Use `@Injectable()` decorator

### Master Database Repository

For entities stored in the master database (no tenant isolation):

```typescript
import { BaseTypeormMasterRepository } from '@/shared/infrastructure/database/typeorm/base-typeorm/base-typeorm-master/base-typeorm-master.repository';
import { TypeormMasterService } from '@/shared/infrastructure/database/typeorm/services/typeorm-master/typeorm-master.service';
import { ExampleAggregate } from '@/example-context/example/domain/aggregates/example.aggregate';
import { ExampleWriteRepository } from '@/example-context/example/domain/repositories/example-write.repository';
import { ExampleTypeormEntity } from '@/example-context/example/infrastructure/database/typeorm/entities/example-typeorm.entity';
import { ExampleTypeormMapper } from '@/example-context/example/infrastructure/database/typeorm/mappers/example-typeorm.mapper';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ExampleTypeormRepository
  extends BaseTypeormMasterRepository<ExampleTypeormEntity>
  implements ExampleWriteRepository
{
  constructor(
    typeormMasterService: TypeormMasterService,
    private readonly exampleTypeormMapper: ExampleTypeormMapper,
  ) {
    super(typeormMasterService, ExampleTypeormEntity);
    this.logger = new Logger(ExampleTypeormRepository.name);
  }

  /**
   * Finds an example by id
   *
   * @param id - The id of the example to find
   * @returns The example if found, null otherwise
   */
  async findById(id: string): Promise<ExampleAggregate | null> {
    this.logger.log(`Finding example by id: ${id}`);
    const entity = await this.repository.findOne({
      where: { id },
    });

    if (!entity) {
      return null;
    }

    return this.exampleTypeormMapper.toDomainEntity(entity);
  }

  /**
   * Saves an example
   *
   * @param example - The example to save
   * @returns The saved example
   */
  async save(example: ExampleAggregate): Promise<ExampleAggregate> {
    this.logger.log(`Saving example: ${example.id.value}`);
    const entity = this.exampleTypeormMapper.toTypeormEntity(example);

    const savedEntity = await this.repository.save(entity);

    return this.exampleTypeormMapper.toDomainEntity(savedEntity);
  }

  /**
   * Deletes an example (soft delete)
   *
   * @param id - The id of the example to delete
   */
  async delete(id: string): Promise<void> {
    this.logger.log(`Soft deleting example by id: ${id}`);
    await this.repository.softDelete(id);
  }
}
```

### Tenant Database Repository

For entities stored in the master database with tenant isolation (using `tenantId` column):

```typescript
import { BaseTypeormTenantRepository } from '@/shared/infrastructure/database/typeorm/base-typeorm/base-typeorm-tenant/base-typeorm-tenant.repository';
import { TypeormMasterService } from '@/shared/infrastructure/database/typeorm/services/typeorm-master/typeorm-master.service';
import { TenantContextService } from '@/shared/infrastructure/services/tenant-context/tenant-context.service';
import { ExampleAggregate } from '@/example-context/example/domain/aggregates/example.aggregate';
import { ExampleWriteRepository } from '@/example-context/example/domain/repositories/example-write.repository';
import { ExampleTypeormEntity } from '@/example-context/example/infrastructure/database/typeorm/entities/example-typeorm.entity';
import { ExampleTypeormMapper } from '@/example-context/example/infrastructure/database/typeorm/mappers/example-typeorm.mapper';
import { Injectable, Logger, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class ExampleTypeormRepository
  extends BaseTypeormTenantRepository<ExampleTypeormEntity>
  implements ExampleWriteRepository
{
  constructor(
    typeormMasterService: TypeormMasterService,
    tenantContextService: TenantContextService,
    private readonly exampleTypeormMapper: ExampleTypeormMapper,
  ) {
    super(typeormMasterService, tenantContextService, ExampleTypeormEntity);
    this.logger = new Logger(ExampleTypeormRepository.name);
  }

  /**
   * Finds an example by id (automatically filtered by tenantId)
   *
   * @param id - The id of the example to find
   * @returns The example if found, null otherwise
   */
  async findById(id: string): Promise<ExampleAggregate | null> {
    // tenantId is automatically added by BaseTypeormTenantRepository
    const entity = await super.findOne({ where: { id } as any });

    if (!entity) {
      return null;
    }

    return this.exampleTypeormMapper.toDomainEntity(entity);
  }

  /**
   * Saves an example (automatically sets tenantId)
   *
   * @param example - The example to save
   * @returns The saved example
   */
  async save(example: ExampleAggregate): Promise<ExampleAggregate> {
    const entity = this.exampleTypeormMapper.toTypeormEntity(example);
    // tenantId is automatically set by saveEntity()
    const saved = await super.saveEntity(entity);
    return this.exampleTypeormMapper.toDomainEntity(saved);
  }

  /**
   * Deletes an example (automatically filtered by tenantId)
   *
   * @param id - The id of the example to delete
   */
  async delete(id: string): Promise<void> {
    // Automatically filters by tenantId
    await super.softDelete(id);
  }
}
```

### Rules

- **Extend base repositories** - Always extend `BaseTypeormMasterRepository` or `BaseTypeormTenantRepository`
- **Use mappers** - Never convert entities to aggregates directly
- **Use soft delete** - Use `softDelete()` instead of `delete()` for soft deletes
- **Log operations** - Use Logger for debugging
- **Request scope for tenant repos** - Use `@Injectable({ scope: Scope.REQUEST })` for tenant repositories

## TypeORM Entities

### Structure

TypeORM entities represent database tables. They must:

- Extend `BaseTypeormEntity` or `BaseTypeormWithTenantEntity`
- Use TypeORM decorators (`@Entity`, `@Column`, `@Index`)
- Use primitive types (not Value Objects)
- Match aggregate structure

### Master Database Entity

```typescript
import { BaseTypeormEntity } from '@/shared/infrastructure/database/typeorm/entities/base-typeorm.entity';
import { ExampleStatusEnum } from '@/shared/domain/enums/example-status/example-status.enum';
import { Column, Entity, Index } from 'typeorm';

@Entity('examples')
@Index(['name'])
export class ExampleTypeormEntity extends BaseTypeormEntity {
  @Column({ type: 'varchar', nullable: true })
  name: string | null;

  @Column({
    type: 'enum',
    enum: ExampleStatusEnum,
  })
  status: ExampleStatusEnum;
}
```

### Tenant Entity

For entities that require tenant isolation:

```typescript
import { BaseTypeormWithTenantEntity } from '@/shared/infrastructure/database/typeorm/entities/base-typeorm-with-tenant.entity';
import { Column, Entity, Index } from 'typeorm';

@Entity('examples')
@Index(['name'])
export class ExampleTypeormEntity extends BaseTypeormWithTenantEntity {
  @Column({ type: 'varchar', nullable: true })
  name: string | null;
}
```

### Base Entity Properties

`BaseTypeormEntity` provides:

- `id: string` (UUID, auto-generated)
- `createdAt: Date` (auto-managed)
- `updatedAt: Date` (auto-managed)
- `deletedAt: Date | null` (for soft deletes)

`BaseTypeormWithTenantEntity` extends `BaseTypeormEntity` and adds:

- `tenantId: string | null` (indexed)

### Rules

- **Extend base entities** - Always extend `BaseTypeormEntity` or `BaseTypeormWithTenantEntity`
- **Use primitive types** - Never use Value Objects in entities
- **Add indexes** - Use `@Index()` for frequently queried columns
- **Use enums** - Use TypeORM enum columns for enum values
- **Nullable columns** - Mark optional fields as `nullable: true`

## TypeORM Mappers

### Structure

TypeORM mappers convert between TypeORM entities and domain aggregates. They must:

- Use `@Injectable()` decorator
- Inject aggregate factories
- Implement `toDomainEntity()` and `toTypeormEntity()` methods

### Mapper

```typescript
import { ExampleAggregate } from '@/example-context/example/domain/aggregates/example.aggregate';
import { ExampleAggregateFactory } from '@/example-context/example/domain/factories/example-aggregate/example-aggregate.factory';
import { ExampleTypeormEntity } from '@/example-context/example/infrastructure/database/typeorm/entities/example-typeorm.entity';
import { ExampleStatusEnum } from '@/shared/domain/enums/example-status/example-status.enum';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ExampleTypeormMapper {
  private readonly logger = new Logger(ExampleTypeormMapper.name);

  constructor(
    private readonly exampleAggregateFactory: ExampleAggregateFactory,
  ) {}

  /**
   * Converts a TypeORM entity to a domain aggregate
   *
   * @param entity - The TypeORM entity to convert
   * @returns The domain aggregate
   */
  toDomainEntity(entity: ExampleTypeormEntity): ExampleAggregate {
    this.logger.log(
      `Converting TypeORM entity to domain entity with id ${entity.id}`,
    );

    return this.exampleAggregateFactory.fromPrimitives({
      id: entity.id,
      name: entity.name ?? null,
      status: entity.status,
      createdAt: entity.createdAt,
      updatedAt: entity.updatedAt,
    });
  }

  /**
   * Converts a domain aggregate to a TypeORM entity
   *
   * @param aggregate - The domain aggregate to convert
   * @returns The TypeORM entity
   */
  toTypeormEntity(aggregate: ExampleAggregate): ExampleTypeormEntity {
    this.logger.log(
      `Converting domain entity with id ${aggregate.id.value} to TypeORM entity`,
    );

    const primitives = aggregate.toPrimitives();

    const entity = new ExampleTypeormEntity();

    entity.id = primitives.id;
    entity.name = primitives.name;
    entity.status = primitives.status as ExampleStatusEnum;
    entity.createdAt = primitives.createdAt;
    entity.updatedAt = primitives.updatedAt;
    entity.deletedAt = null;

    return entity;
  }
}
```

### Rules

- **Use factories** - Always use aggregate factories to create aggregates
- **Convert primitives** - Use `aggregate.toPrimitives()` to get primitives
- **Handle nulls** - Use `?? null` for nullable fields
- **Reset deletedAt** - Always set `deletedAt = null` when creating new entities
- **Log conversions** - Use Logger for debugging

## MongoDB Read Repositories

### Structure

MongoDB repositories implement read repository interfaces and handle queries of view models from MongoDB. They must:

- Extend `BaseMongoMasterRepository` or `BaseMongoTenantRepository`
- Implement the domain read repository interface
- Use mappers to convert between DTOs and view models
- Use `@Injectable()` decorator

### Master Database Repository

```typescript
import { Criteria } from '@/shared/domain/entities/criteria';
import { PaginatedResult } from '@/shared/domain/entities/paginated-result.entity';
import { BaseMongoMasterRepository } from '@/shared/infrastructure/database/mongodb/base-mongo/base-mongo-master/base-mongo-master.repository';
import { MongoMasterService } from '@/shared/infrastructure/database/mongodb/services/mongo-master/mongo-master.service';
import { ExampleReadRepository } from '@/example-context/example/domain/repositories/example-read.repository';
import { ExampleViewModel } from '@/example-context/example/domain/view-models/example.view-model';
import { ExampleMongoDBMapper } from '@/example-context/example/infrastructure/database/mongodb/mappers/example-mongodb.mapper';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ExampleMongoRepository
  extends BaseMongoMasterRepository
  implements ExampleReadRepository
{
  private readonly collectionName = 'examples';

  constructor(
    mongoMasterService: MongoMasterService,
    private readonly exampleMongoDBMapper: ExampleMongoDBMapper,
  ) {
    super(mongoMasterService);
    this.logger = new Logger(ExampleMongoRepository.name);
  }

  /**
   * Finds an example by id
   *
   * @param id - The id of the example to find
   * @returns The example if found, null otherwise
   */
  async findById(id: string): Promise<ExampleViewModel | null> {
    this.logger.log(`Finding example by id: ${id}`);

    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );
    const doc = await collection.findOne({ id });

    return doc
      ? this.exampleMongoDBMapper.toViewModel({
          id: doc.id,
          name: doc.name,
          status: doc.status,
          createdAt: doc.createdAt,
          updatedAt: doc.updatedAt,
        })
      : null;
  }

  /**
   * Finds examples by criteria
   *
   * @param criteria - The criteria to find examples by
   * @returns The examples found
   */
  async findByCriteria(
    criteria: Criteria,
  ): Promise<PaginatedResult<ExampleViewModel>> {
    this.logger.log(
      `Finding examples by criteria: ${JSON.stringify(criteria)}`,
    );

    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );

    // 01: Build MongoDB query from criteria
    const mongoQuery = this.buildMongoQuery(criteria);
    const sortQuery = this.buildSortQuery(criteria);

    // 02: Calculate pagination
    const page = criteria.pagination.page || 1;
    const limit = criteria.pagination.perPage || 10;
    const skip = (page - 1) * limit;

    // 03: Execute query with pagination
    const [data, total] = await Promise.all([
      collection
        .find(mongoQuery)
        .sort(sortQuery)
        .skip(skip)
        .limit(limit)
        .toArray(),
      collection.countDocuments(mongoQuery),
    ]);

    // 04: Convert MongoDB documents to view models
    const examples = data.map((doc) =>
      this.exampleMongoDBMapper.toViewModel({
        id: doc.id,
        name: doc.name,
        status: doc.status,
        createdAt: doc.createdAt,
        updatedAt: doc.updatedAt,
      }),
    );

    return new PaginatedResult<ExampleViewModel>(examples, total, page, limit);
  }

  /**
   * Saves an example view model (upsert operation)
   *
   * @param viewModel - The example view model to save
   */
  async save(viewModel: ExampleViewModel): Promise<void> {
    this.logger.log(`Saving example view model with id: ${viewModel.id}`);

    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );
    const mongoData = this.exampleMongoDBMapper.toMongoData(viewModel);

    // 01: Use upsert to either insert or update the view model
    await collection.replaceOne({ id: viewModel.id }, mongoData, {
      upsert: true,
    });
  }

  /**
   * Deletes an example view model by id
   *
   * @param id - The id of the example view model to delete
   */
  async delete(id: string): Promise<void> {
    this.logger.log(`Deleting example view model by id: ${id}`);

    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );

    await collection.deleteOne({ id });
  }
}
```

### Rules

- **Extend base repositories** - Always extend `BaseMongoMasterRepository` or `BaseMongoTenantRepository`
- **Use mappers** - Never convert DTOs to view models directly
- **Use upsert for save** - Use `replaceOne` with `upsert: true` for save operations
- **Use buildMongoQuery** - Use base repository methods to build queries
- **Log operations** - Use Logger for debugging

## MongoDB DTOs

### Structure

MongoDB DTOs represent the structure of documents stored in MongoDB. They must:

- Be type aliases (not interfaces or classes)
- Use primitive types only
- Match view model structure

### DTO

```typescript
export type ExampleMongoDbDto = {
  id: string;
  name: string | null;
  status: string;
  createdAt: Date;
  updatedAt: Date;
};
```

### Rules

- **Type aliases** - Use `type` not `interface` or `class`
- **Primitive types only** - Never use Value Objects or ViewModels
- **Match view model** - DTO structure should mirror view model structure
- **Include timestamps** - Always include `createdAt` and `updatedAt`

## MongoDB Mappers

### Structure

MongoDB mappers convert between MongoDB DTOs and domain view models. They must:

- Use `@Injectable()` decorator
- Inject view model factories
- Implement `toViewModel()` and `toMongoData()` methods

### Mapper

```typescript
import { ExampleViewModelFactory } from '@/example-context/example/domain/factories/example-view-model/example-view-model.factory';
import { ExampleViewModel } from '@/example-context/example/domain/view-models/example.view-model';
import { ExampleMongoDbDto } from '@/example-context/example/infrastructure/database/mongodb/dtos/example-mongodb.dto';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ExampleMongoDBMapper {
  private readonly logger = new Logger(ExampleMongoDBMapper.name);

  constructor(
    private readonly exampleViewModelFactory: ExampleViewModelFactory,
  ) {}

  /**
   * Converts a MongoDB document to a view model
   *
   * @param doc - The MongoDB document to convert
   * @returns The view model
   */
  toViewModel(doc: ExampleMongoDbDto): ExampleViewModel {
    this.logger.log(
      `Converting MongoDB document to view model with id ${doc.id}`,
    );

    return this.exampleViewModelFactory.create({
      id: doc.id,
      name: doc.name,
      status: doc.status,
      createdAt: new Date(doc.createdAt),
      updatedAt: new Date(doc.updatedAt),
    });
  }

  /**
   * Converts a view model to a MongoDB document
   *
   * @param viewModel - The view model to convert
   * @returns The MongoDB document
   */
  toMongoData(viewModel: ExampleViewModel): ExampleMongoDbDto {
    this.logger.log(
      `Converting view model with id ${viewModel.id} to MongoDB document`,
    );

    return {
      id: viewModel.id,
      name: viewModel.name,
      status: viewModel.status,
      createdAt: viewModel.createdAt,
      updatedAt: viewModel.updatedAt,
    };
  }
}
```

### Rules

- **Use factories** - Always use view model factories to create view models
- **Convert dates** - Convert Date objects when reading from MongoDB
- **Log conversions** - Use Logger for debugging

## Best Practices

### ✅ Good Practices

- Extend base repositories (`BaseTypeormMasterRepository`, `BaseMongoMasterRepository`)
- Use mappers for all conversions
- Use factories to create aggregates and view models
- Use soft delete for TypeORM entities
- Use upsert for MongoDB save operations
- Log all operations
- Use request scope for tenant repositories
- Add indexes for frequently queried columns

### ❌ Bad Practices

- Creating repositories without extending base classes
- Converting entities/DTOs to aggregates/view models directly (should use mappers)
- Using hard delete instead of soft delete
- Not using factories to create domain objects
- Not handling null values properly
- Not logging operations
- Using domain types in infrastructure entities/DTOs

## Examples

### ✅ Good: TypeORM Repository

```typescript
@Injectable()
export class ExampleTypeormRepository
  extends BaseTypeormMasterRepository<ExampleTypeormEntity>
  implements ExampleWriteRepository
{
  constructor(
    typeormMasterService: TypeormMasterService,
    private readonly exampleTypeormMapper: ExampleTypeormMapper,
  ) {
    super(typeormMasterService, ExampleTypeormEntity);
    this.logger = new Logger(ExampleTypeormRepository.name);
  }

  async findById(id: string): Promise<ExampleAggregate | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.exampleTypeormMapper.toDomainEntity(entity) : null;
  }

  async save(example: ExampleAggregate): Promise<ExampleAggregate> {
    const entity = this.exampleTypeormMapper.toTypeormEntity(example);
    const saved = await this.repository.save(entity);
    return this.exampleTypeormMapper.toDomainEntity(saved);
  }
}
```

### ❌ Bad: TypeORM Repository

```typescript
@Injectable()
export class ExampleTypeormRepository {
  // ❌ Should extend BaseTypeormMasterRepository
  constructor(private readonly repository: Repository<ExampleTypeormEntity>) {}

  async findById(id: string): Promise<ExampleAggregate> {
    const entity = await this.repository.findOne({ where: { id } });
    return new ExampleAggregate(entity); // ❌ Should use mapper and factory
  }

  async save(example: ExampleAggregate): Promise<ExampleAggregate> {
    const entity = {
      id: example.id.value, // ❌ Should use mapper
      name: example.name.value,
    };
    await this.repository.save(entity);
    return example; // ❌ Should convert back using mapper
  }
}
```

### ✅ Good: MongoDB Repository

```typescript
@Injectable()
export class ExampleMongoRepository
  extends BaseMongoMasterRepository
  implements ExampleReadRepository
{
  private readonly collectionName = 'examples';

  constructor(
    mongoMasterService: MongoMasterService,
    private readonly exampleMongoDBMapper: ExampleMongoDBMapper,
  ) {
    super(mongoMasterService);
  }

  async findById(id: string): Promise<ExampleViewModel | null> {
    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );
    const doc = await collection.findOne({ id });
    return doc ? this.exampleMongoDBMapper.toViewModel(doc) : null;
  }

  async save(viewModel: ExampleViewModel): Promise<void> {
    const collection = this.mongoMasterService.getCollection(
      this.collectionName,
    );
    const mongoData = this.exampleMongoDBMapper.toMongoData(viewModel);
    await collection.replaceOne({ id: viewModel.id }, mongoData, {
      upsert: true,
    });
  }
}
```

### ❌ Bad: MongoDB Repository

```typescript
@Injectable()
export class ExampleMongoRepository {
  // ❌ Should extend BaseMongoMasterRepository
  async findById(id: string): Promise<ExampleViewModel> {
    const doc = await this.collection.findOne({ id });
    return new ExampleViewModel(doc); // ❌ Should use mapper and factory
  }

  async save(viewModel: ExampleViewModel): Promise<void> {
    await this.collection.insertOne(viewModel); // ❌ Should use mapper and upsert
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sisques-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
