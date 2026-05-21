## nestjs-boilerplate

> You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns.


You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns.

Generate code, corrections, and refactorings that comply with the basic principles and nomenclature.

## TypeScript General Guidelines

### Basic TypeScript Principles

- Use English for all code and documentation.
- Always declare the type of each variable and function (parameters and return value).
  - Avoid using any.
  - Create necessary types.
- Use JSDoc to document public classes and methods.
- Don't leave blank lines within a function.
- One export per file.

### Exceptions

- Use exceptions to handle errors you don't expect.
- If you catch an exception, it should be to:
  - Fix an expected problem.
  - Add context.
  - Otherwise, use a global handler.

### Testing

- Follow the Arrange-Act-Assert convention for tests.
- Name test variables clearly.
  - Follow the convention: inputX, mockX, actualX, expectedX, etc.
- Write unit tests for each public function.
  - Use test doubles to simulate dependencies.
    - Except for third-party dependencies that are not expensive to execute.
- Write acceptance tests for each module.
  - Follow the Given-When-Then convention.

## NestJS Guidelines

- Let generate code following sample code and follow the same structure and principles.

### Basic NestJS Principles

- Use modular architecture
- Encapsulate the API in modules.
  - One module per main domain/route.
  - One controller for its route.
    - And other controllers for secondary routes.
  - A models folder with data types.
    - DTOs validated with class-validator for inputs.
    - Declare simple types for outputs.
  - A services module with business logic and persistence.
    - Entities with MikroORM for data persistence.
    - One service per entity.
- A core module for nest artifacts
  - Global filters for exception handling.
  - Global middlewares for request management.
  - Guards for permission management.
  - Interceptors for request management.
- A shared module for services shared between modules.
  - Utilities
  - Shared business logic

### Prisma

- Use the Prisma library for database operations.
- Design a system where many users can connect together and share the same profile. Most business models will relate to the profile instead of the user.
- Schema:
  - id is a BigInt, auto-incremented.
  - createdAt and updatedAt are DateTime, auto-generated.
  - profileId is a BigInt, optional, related to the profile.

### NestJS Libraries

- Create a library for each domain.
  - script: nest g lib <library-name>
- Each library should includes:
  - entities directory: defines rules and swagger for database entities.
  - dtos directory: defines data transfer objects.
  - models directory: defines custom model classes.

### Entities

- Should place in the entities directory and be named like the entity name.
- Convention: entity-name.entity.ts
- Should be a class that implements the entity interface.
- For relational entities, read the prisma.schema file and define the relations at the bottom of the entity class (under the // Relations section).
- By default, all properties should have the "Expose" decorator and the "ApiProperty" decorator.
- No need to validator decorators.
- Let reference the following example:

```typescript
import { Expose, Type } from 'class-transformer'
import { ApiProperty } from '@nestjs/swagger'
import { ProfileEntity } from '@app/profile/entities/profile.entity'

export class TodoEntity {
  @ApiProperty()
  @Expose()
  id: bigint

  @ApiProperty()
  @Expose()
  createdAt: Date

  @ApiProperty()
  @Expose()
  updatedAt: Date

  @ApiProperty()
  @Expose()
  title: string

  @ApiProperty()
  @Expose()
  description?: string

  @ApiProperty()
  @Expose()
  done: boolean

  @ApiProperty()
  @Expose()
  profileId: bigint

  @ApiProperty({ type: () => ProfileEntity })
  @Type(() => ProfileEntity)
  @Expose()
  profile: ProfileEntity
}
```

### DTOs

- Utilize class type support from @nestjs/swagger (e.g., PickType, PartialType).
- The class-validator and class-transformer decorators will be lost if overridden in a class. Therefore, we must define all decorators in the DTO, even if they are already defined in the base class.
- Create Entity DTO:
  - Convention: create-entity-name.dto.ts
  - No need profileId field.
  - Include validation decorators.
  - Include ApiProperty decorators.
  - Let reference the following example:

```typescript
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger'
import { Expose } from 'class-transformer'
import { IsNotEmpty, IsOptional, IsString } from 'class-validator'

export class CreateTodoDto {
  @ApiProperty()
  @IsString()
  @IsNotEmpty()
  @Expose()
  title: string

  @ApiPropertyOptional()
  @IsString()
  @IsOptional()
  @Expose()
  description?: string
}
```

- Update Entity DTO:
  - Convention: edit-entity-name.dto.ts
  - Pick the fields that can be updated from CreateXXXDto to _UpdateXXXDto via PickType.
  - Use PartialType to make the fields optional.
  - Let reference the following example:

```typescript
import { PartialType, PickType } from '@nestjs/swagger'
import { CreateTodoDto } from './create-todo.dto'

class _UpdateTodoDto extends PickType(CreateTodoDto, ['title', 'description']) {}
export class UpdateTodoDto extends PartialType(_UpdateTodoDto) {}
```

- Query Entity DTO:
  - Convention: query-entity-name.dto.ts
  - Number fields should be annotated with @Type(() => Number)
  - Let reference the following example:

```typescript
import { IsIncludeOnlyKeys, IsIncludeOnlyValues } from '@app/helper/class.validator'
import { ApiPropertyOptional } from '@nestjs/swagger'
import { Prisma } from '@prisma/client'
import { Expose, Transform, Type } from 'class-transformer'
import { IsOptional, IsJSON, IsNumber, IsObject, ValidateNested, IsArray } from 'class-validator'

const TodoFields = Object.values(Prisma.TodoScalarFieldEnum)

export class QueryTodoDto {
  @ApiPropertyOptional()
  @IsOptional()
  @IsObject()
  @IsIncludeOnlyKeys(TodoFields)
  @Expose()
  where?: Record<string, any>

  @ApiPropertyOptional({ type: () => Map })
  @IsOptional()
  @IsObject()
  @IsIncludeOnlyKeys(TodoFields)
  @IsIncludeOnlyValues(Object.values(Prisma.SortOrder))
  @Expose()
  sort?: Record<string, string>

  @ApiPropertyOptional({ type: () => String, isArray: true })
  @IsOptional()
  @IsArray()
  @IsIncludeOnlyKeys(TodoFields)
  @Expose()
  select?: string[]

  @ApiPropertyOptional({ type: () => String, isArray: true })
  @IsOptional()
  @IsArray()
  @IsIncludeOnlyKeys(['profile'])
  @Expose()
  include?: string[]

  @ApiPropertyOptional()
  @IsOptional()
  @IsNumber()
  @Type(() => Number)
  @Expose()
  skip?: number

  @ApiPropertyOptional()
  @IsOptional()
  @IsNumber()
  @Type(() => Number)
  @Expose()
  take?: number
}
```

- Response DTO:
  - Convention: xxx.res.dto.ts

### Controllers

- Let reference the following example:

```typescript
import { ParseBigIntPipe } from '@app/core/pipes/parse-bigint.pipe'
import { CreateTodoDto } from './dtos/create-todo.dto'
import { TodoService } from './todo.service'

import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  Post,
  Put,
  Query,
  UseGuards,
  UseInterceptors,
  UsePipes,
  ValidationPipe,
} from '@nestjs/common'
import { UpdateTodoDto } from './dtos/update-todo.dto'
import { QueryTodoDto } from './dtos/query-todo.dto'
import { ApiBearerAuth, ApiOkResponse } from '@nestjs/swagger'
import { JwtGuard } from '@app/auth/guards/jwt.guard'
import { TodoEntity } from './entities/todo.entity'
import { CacheTTL } from '@nestjs/cache-manager'
import { AppCacheInterceptor } from '@app/core/interceptors/app-cache-interceptor'
import { AppCacheKey } from '@app/core/decorators/app-cache-key.decorator'
import { CurUser } from '@app/core/decorators/user.decorator'
import { User } from '@prisma/client'
import { RawQuery } from '@app/core/decorators/query.decorator'

@Controller('todo')
export class TodoController {
  constructor(private readonly todoService: TodoService) {}

  @Get()
  @ApiBearerAuth()
  @ApiOkResponse({ type: () => TodoEntity, isArray: true })
  @UseGuards(JwtGuard)
  @CacheTTL(2000)
  @UseInterceptors(AppCacheInterceptor)
  getTodos(@RawQuery() queryTodoDto: QueryTodoDto) {
    return this.todoService.getTodos(queryTodoDto)
  }

  @Get(':id')
  @ApiBearerAuth()
  @ApiOkResponse({ type: () => TodoEntity })
  @UseGuards(JwtGuard)
  @CacheTTL(2000)
  @AppCacheKey((req) => `todo-${req.params.id}`)
  @UseInterceptors(AppCacheInterceptor)
  getTodo(@Param('id', ParseBigIntPipe) id: bigint) {
    return this.todoService.getTodo(id)
  }

  @Post()
  @ApiBearerAuth()
  @ApiOkResponse({ type: () => TodoEntity })
  @UseGuards(JwtGuard)
  createTodo(@Body() createTodoDto: CreateTodoDto, @CurUser() user: User) {
    return this.todoService.createTodo(createTodoDto, user)
  }

  @Put(':id')
  @ApiBearerAuth()
  @ApiOkResponse({ type: () => TodoEntity })
  @UseGuards(JwtGuard)
  updateTodo(@Param('id', ParseBigIntPipe) id: bigint, @Body() updateTodoDto: UpdateTodoDto, @CurUser() user: User) {
    return this.todoService.updateTodo(id, updateTodoDto, user)
  }

  @Delete(':id')
  @ApiBearerAuth()
  @ApiOkResponse({ type: () => TodoEntity })
  @UseGuards(JwtGuard)
  deleteTodo(@Param('id', ParseBigIntPipe) id: bigint, @CurUser() user: User) {
    return this.todoService.deleteTodo(id, user)
  }
}
```

### Services

- Let reference the following example:
  - "create", "update", "delete" functions include user parameter

```typescript
import { Injectable } from '@nestjs/common'
import { PrismaService } from 'nestjs-prisma'
import { CreateTodoDto } from './dtos/create-todo.dto'
import { UpdateTodoDto } from './dtos/update-todo.dto'
import { QueryTodoDto } from './dtos/query-todo.dto'
import { User } from '@prisma/client'
import { th } from '@app/helper/transform.helper'
import { TodoEntity } from './entities/todo.entity'

@Injectable()
export class TodoService {
  constructor(private readonly prisma: PrismaService) {}

  async getTodos(queryTodoDto: QueryTodoDto) {
    const { select, include } = queryTodoDto
    const todos = await this.prisma.todo.findMany({
      where: queryTodoDto.where,
      orderBy: queryTodoDto.sort,
      take: queryTodoDto.take,
      skip: queryTodoDto.skip,
      ...(select
        ? { select: Object.fromEntries(select.map((key) => [key, true])) }
        : include
          ? { include: Object.fromEntries(include.map((key) => [key, true])) }
          : {}),
    })
    return th.toInstancesSafe(TodoEntity, todos)
  }

  async getTodo(id: bigint) {
    const todo = await this.prisma.todo.findUniqueOrThrow({
      where: { id },
    })
    return th.toInstanceSafe(TodoEntity, todo)
  }

  async createTodo(dto: CreateTodoDto, user: User) {
    const todo = await this.prisma.todo.create({
      data: {
        ...dto,
        profileId: user.profileId,
      },
    })
    return th.toInstanceSafe(TodoEntity, todo)
  }

  async updateTodo(id: bigint, dto: UpdateTodoDto, user: User) {
    const todo = await this.prisma.todo.update({
      where: { id, profileId: user.profileId },
      data: dto,
    })
    return th.toInstanceSafe(TodoEntity, todo)
  }

  async deleteTodo(id: bigint, user: User) {
    await this.prisma.todo.delete({
      where: { id, profileId: user.profileId },
    })
  }
}
```

### Testing

- Use the standard Jest framework for testing.
- Write tests for each controller and service.
- Write end to end tests for each api module.
- Add a admin/test method to each controller as a smoke test.
- Let reference the following example:

```typescript
import { TodoService } from './todo.service'
import { TestContext, testHelper, UserContextTestType } from '@app/spec/test.helper'
import { INestApplication } from '@nestjs/common'
import { PrismaService } from 'nestjs-prisma'
import { CreateTodoDto } from './dtos/create-todo.dto'
import { TodoModule } from './todo.module'
import qs from 'qs'
import { Todo } from '@prisma/client'
import { UpdateTodoDto } from './dtos/update-todo.dto'

describe('TodoSpec', () => {
  let tc: TestContext
  let app: INestApplication
  let prismaService: PrismaService
  let uc: UserContextTestType

  beforeAll(async () => {
    tc = await testHelper.createContext({
      imports: [TodoModule],
    })
    app = tc.app
    prismaService = app.get(PrismaService)
    uc = await tc.generateAcount()
  })

  afterAll(async () => await tc?.clean())

  describe('Create', () => {
    test('Create:TitleIsRequired', async () => {
      const res = await uc.request((r) => r.post('/todo')).send({} as CreateTodoDto)
      expect(res).toBeBad(/title should not be empty/)
    })
    test('Create:Success', async () => {
      const res = await uc.request((r) => r.post('/todo')).send({ title: 'Test Todo' } as CreateTodoDto)
      expect(res).toBeCreated()
      expect(res.body.title).toBe('Test Todo')
    })
  })

  describe('Fetch', () => {
    let todo: Todo
    beforeAll(async () => {
      const res = await uc.request((r) => r.post('/todo')).send({ title: 'Test Todo' } as CreateTodoDto)
      todo = res.body
    })
    test('GetTodo', async () => {
      const res = await uc.request((r) => r.get(`/todo/${todo.id}`))
      expect(res).toBeOK()
      expect(res.body.id).toBe(todo.id)
    })
    test('GetTodos', async () => {
      const paramDto = {
        where: {
          id: {
            gt: 0,
          },
          title: {
            startsWith: 'Test',
          },
        },
        // select: ['title'],
        include: ['profile'],
        take: 3,
      }
      const param = qs.stringify(paramDto)
      const res = await uc.request((r) => r.get(`/todo?${param}`))
      expect(res).toBeOK()
      expect(res.body.length).toBeGreaterThan(0)
    })
  })
  describe('Update', () => {
    let todo: Todo
    beforeAll(async () => {
      const res = await uc.request((r) => r.post('/todo')).send({ title: 'Test Todo' } as CreateTodoDto)
      todo = res.body
    })
    test('Update:Success', async () => {
      const res = await uc.request((r) => r.put(`/todo/${todo.id}`)).send({ title: 'Updated Todo' } as UpdateTodoDto)
      expect(res).toBeOK()
      const newTodoRes = await uc.request((r) => r.get(`/todo/${todo.id}`))
      expect(newTodoRes.body.title).toBe('Updated Todo')
    })
  })
  describe('Delete', () => {
    let todo: Todo
    beforeAll(async () => {
      const res = await uc.request((r) => r.post('/todo')).send({ title: 'Test Todo' } as CreateTodoDto)
      todo = res.body
    })
    test('Delete:Success', async () => {
      const res = await uc.request((r) => r.delete(`/todo/${todo.id}`))
      expect(res).toBeOK()
      const newTodoRes = await uc.request((r) => r.get(`/todo/${todo.id}`))
      expect(newTodoRes).toBe404()
    })
  })
})
```

---
> Source: [cyberk-lab/nestjs-boilerplate](https://github.com/cyberk-lab/nestjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
