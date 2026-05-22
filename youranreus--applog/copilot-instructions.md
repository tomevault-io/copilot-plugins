## backend

> - **框架**: NestJS 10.x + Fastify

# Backend (NestJS) 专用规则

## 技术栈
- **框架**: NestJS 10.x + Fastify
- **数据库**: TypeORM + MySQL2
- **验证**: class-validator + class-transformer
- **语言**: TypeScript 5.x (严格模式)
- **包管理**: pnpm workspace

## 后端 TypeScript 配置
- Target: ES2021
- Module: CommonJS
- 路径别名: `@/*` 映射到 `src/*`
- 装饰器支持: 启用 `experimentalDecorators` 和 `emitDecoratorMetadata`

## 快速参考

### 核心装饰器和工具（来自 @reus-able/nestjs）
```typescript
// 权限控制
import { AuthRoles, UserParams } from '@reus-able/nestjs';
@AuthRoles('user')    // 需要 user 权限
@AuthRoles('admin')   // 需要 admin 权限
@UserParams() user: UserJwtPayload  // 获取当前用户

// 日志记录
import { HLogger, HLOGGER_TOKEN } from '@reus-able/nestjs';
@Inject(HLOGGER_TOKEN) private logger: HLogger;

// 异常处理
import { BusinessException } from '@reus-able/nestjs';
throw new BusinessException('错误信息');
```

### 全局模块（已配置，直接使用）
- **LoggerModule**: 日志服务（通过 `HLOGGER_TOKEN` 注入）
- **ConfigModule**: 配置管理（通过 `ConfigService` 注入）
- **AuthGuard**: 权限守卫（自动检查 `@AuthRoles`）
- **ValidationPipe**: 参数校验（自动转换类型）
- **异常过滤器**: 统一错误响应格式

### 类型定义（来自 @reus-able/types）
```typescript
import type { UserJwtPayload } from '@reus-able/types';
```

## 项目基础设施（必读）

### 全局模块

**LoggerModule** - 来自 `@reus-able/nestjs`
```typescript
import { HLogger, HLOGGER_TOKEN } from '@reus-able/nestjs';

@Injectable()
export class YourService {
  @Inject(HLOGGER_TOKEN)
  private logger: HLogger;

  someMethod() {
    this.logger.log('message', YourService.name);
    this.logger.error('error', YourService.name);
  }
}
```

**ConfigModule** - 来自 `@nestjs/config`
- 已配置为全局模块
- 支持多环境配置：`.env.production.local`, `.env.development.local`, `.env.production`, `.env.development`, `.env`
- 通过依赖注入使用：
```typescript
import { ConfigService } from '@nestjs/config';

constructor(private config: ConfigService) {}

const value = this.config.get<string>('KEY_NAME', 'default_value');
```

**TypeORM + MySQL**
- 已配置全局 TypeORM 连接
- 新模块需要注册实体：
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([YourEntity])],
})
```

### 全局守卫与拦截器（已配置）
- **AuthGuard**: 全局权限守卫，自动检查路由权限
- **TransformInterceptor**: 统一响应格式转换
- **AllExceptionsFilter & HttpExceptionFilter**: 统一异常处理
- **ValidationPipe**: 全局参数校验（支持自动类型转换）

### 权限控制装饰器

**@AuthRoles** - 路由权限控制
```typescript
import { AuthRoles, UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Controller('api')
export class SomeController {
  // 需要 user 权限（普通用户）
  @Get('data')
  @AuthRoles('user')
  getData(@UserParams() user: UserJwtPayload) {
    return this.service.getData(user.id);
  }

  // 需要 admin 权限（管理员）
  @Post('admin/action')
  @AuthRoles('admin')
  adminAction() {
    return this.service.doAdmin();
  }

  // 不需要权限校验时，不使用装饰器
  @Get('public')
  publicData() {
    return this.service.getPublic();
  }
}
```

**权限等级说明**：
- 项目只有两种权限：`user`（普通用户）和 `admin`（管理员）
- 不需要权限校验的接口不要使用 `@AuthRoles` 装饰器
- 使用 `@UserParams()` 装饰器获取当前登录用户信息

**@UserParams** - 获取当前用户信息
```typescript
import { UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Get('profile')
@AuthRoles('user')
getUserProfile(@UserParams() user: UserJwtPayload) {
  // user 包含: id, username, role 等信息
  return this.service.getProfile(user.id);
}
```

### 异常处理
优先使用 `BusinessException` 处理业务异常：
```typescript
import { BusinessException } from '@reus-able/nestjs';

if (!data) {
  throw new BusinessException('数据不存在');
}

// 也可以使用 NestJS 内置异常
throw new NotFoundException('资源未找到');
```

### API 版本控制
项目启用了 URI 版本控制，默认版本为 `VERSION_NEUTRAL` 和 `'1'`：
```typescript
import { Controller, VERSION_NEUTRAL } from '@nestjs/common';

@Controller({
  path: 'users',
  version: [VERSION_NEUTRAL, '1'],
})
export class UserController {}
```

## NestJS 开发规范

### 1. 模块组织
- 按功能模块划分代码（如 `user`, `post`, `auth` 等）
- 每个模块包含: `*.module.ts`, `*.controller.ts`, `*.service.ts`
- 使用 barrel exports（index.ts）统一导出
- 模块中使用实体时，必须在 Module 的 imports 中注册：`TypeOrmModule.forFeature([Entity])`

### 2. 控制器 (Controllers)
```typescript
import { Controller, Get, Post, Param, Body, VERSION_NEUTRAL } from '@nestjs/common';
import { AuthRoles, UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Controller({
  path: 'users',
  version: [VERSION_NEUTRAL, '1'],
})
export class UserController {
  constructor(private readonly userService: UserService) {}

  // 公开接口，无需权限
  @Get('list')
  async getList(): Promise<UserResponseDto[]> {
    return this.userService.getList();
  }

  // 需要 user 权限
  @Get(':id')
  @AuthRoles('user')
  async findOne(
    @Param('id') id: string,
    @UserParams() user: UserJwtPayload,
  ): Promise<UserResponseDto> {
    return this.userService.findOne(id, user);
  }

  // 需要 admin 权限
  @Post('create')
  @AuthRoles('admin')
  async create(
    @Body() createDto: CreateUserDto,
    @UserParams() user: UserJwtPayload,
  ): Promise<UserResponseDto> {
    return this.userService.create(createDto, user);
  }
}
```
- **必须使用 API 版本控制**：`version: [VERSION_NEUTRAL, '1']`
- **权限控制**：根据接口需求使用 `@AuthRoles('user')` 或 `@AuthRoles('admin')`
- **获取当前用户**：使用 `@UserParams()` 装饰器
- 使用 DTO 进行请求/响应类型定义
- 控制器应保持轻量，业务逻辑放在 Service 中

### 3. 服务 (Services)
```typescript
import { Inject, Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { ConfigService } from '@nestjs/config';
import { HLogger, HLOGGER_TOKEN, BusinessException } from '@reus-able/nestjs';

@Injectable()
export class UserService {
  @InjectRepository(User)
  private userRepository: Repository<User>;

  @Inject(HLOGGER_TOKEN)
  private logger: HLogger;

  constructor(private config: ConfigService) {}

  private log(message: string) {
    this.logger.log(message, UserService.name);
  }

  private error(message: string) {
    this.logger.error(message, UserService.name);
  }

  /**
   * 根据 ID 查找用户
   * @param id - 用户 ID
   * @returns 用户实体
   * @throws {BusinessException} 当用户不存在时抛出异常
   * 
   * 逻辑说明：
   * 1. 记录查询日志
   * 2. 从数据库查询用户
   * 3. 若用户不存在，记录错误并抛出业务异常
   * 4. 返回用户实体
   */
  async findOne(id: string): Promise<User> {
    this.log(`查找用户: ${id}`);
    
    // 查询数据库
    const user = await this.userRepository.findOne({ where: { id } });
    
    // 用户不存在时抛出异常
    if (!user) {
      this.error(`用户不存在: ${id}`);
      throw new BusinessException(`用户 #${id} 不存在`);
    }
    
    return user;
  }

  /**
   * 创建新用户
   * @param data - 创建用户的 DTO
   * @returns 创建成功的用户实体
   * 
   * 逻辑说明：
   * 1. 从配置中获取 API URL
   * 2. 记录创建用户日志
   * 3. 保存用户到数据库
   * 4. 返回创建的用户实体
   */
  async create(data: CreateUserDto): Promise<User> {
    // 获取配置
    const apiUrl = this.config.get<string>('API_URL', 'http://localhost');
    this.log(`创建用户，API: ${apiUrl}`);
    
    // 保存用户到数据库
    return await this.userRepository.save(data);
  }
}
```
- 使用 `@Injectable()` 装饰器
- **必须注入并使用 Logger**：`@Inject(HLOGGER_TOKEN) private logger: HLogger`
- **使用 ConfigService** 读取配置：通过构造函数注入
- **优先使用 BusinessException** 处理业务异常
- 依赖注入通过构造函数或属性注入
- 为关键操作添加日志记录
- **必须添加 JSDoc 注释**：标注参数、返回值和逻辑说明

### 4. DTO (Data Transfer Objects)
```typescript
import { IsString, IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  @IsNotEmpty()
  email: string;
}
```
- 使用 `class-validator` 装饰器进行验证
- 创建独立的 DTO 文件
- 区分 CreateDto、UpdateDto、ResponseDto

### 5. Entity (实体)
```typescript
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'varchar', length: 255 })
  name: string;

  @Column({ type: 'varchar', length: 255, unique: true })
  email: string;

  @CreateDateColumn()
  createdAt: Date;
}
```
- 使用 TypeORM 装饰器定义实体
- 明确指定数据库表名和列类型
- 使用合适的关系装饰器（@OneToMany, @ManyToOne 等）

### 6. 错误处理
- **优先使用 `BusinessException`**（来自 `@reus-able/nestjs`）处理业务异常
- 也可使用 NestJS 内置异常类（NotFoundException, BadRequestException 等）
- 创建自定义异常时继承 `HttpException`
- 项目已配置全局异常过滤器（AllExceptionsFilter, HttpExceptionFilter），会自动统一处理错误响应

```typescript
import { BusinessException } from '@reus-able/nestjs';
import { NotFoundException } from '@nestjs/common';

// 推荐：业务异常使用 BusinessException
if (!user) {
  throw new BusinessException('用户不存在');
}

// 也可以：使用 NestJS 内置异常
if (!resource) {
  throw new NotFoundException('资源未找到');
}
```

## 后端开发重点

### 必须使用的装饰器和工具
- `@AuthRoles('user')` / `@AuthRoles('admin')` - 权限控制
- `@UserParams()` - 获取当前用户
- `@Inject(HLOGGER_TOKEN) private logger: HLogger` - 日志记录
- `BusinessException` - 业务异常处理

### 模块结构
- 每个模块包含: `*.module.ts`, `*.controller.ts`, `*.service.ts`
- 使用 `TypeOrmModule.forFeature([Entity])` 注册实体
- Controller 必须指定 API 版本: `version: [VERSION_NEUTRAL, '1']`

### 文件命名
- **实体、DTO、类文件**: PascalCase（如 `User.ts`, `CreateUserDto.ts`）
- **模块文件**: kebab-case（如 `user.controller.ts`, `user.service.ts`）

### 代码规范
- 所有 Service 必须注入并使用 `HLogger`
- 使用 `ConfigService` 读取配置，不要硬编码
- 所有函数必须添加 JSDoc 注释
- 单函数不超过 150 行
- 使用 `async/await`，禁止混用 `.then()`

### 数据库操作
- 使用 TypeORM Query Builder 处理复杂查询
- 使用事务处理关联操作
- 注意 N+1 查询问题，合理使用 relations
- 新模块必须在 Module 中注册实体：`TypeOrmModule.forFeature([Entity])`

### API 设计
- 遵循 RESTful 规范
- 使用合适的 HTTP 状态码
- 使用 `BusinessException` 提供清晰的错误信息
- 合理设置接口权限（user / admin / public）

## 最佳实践

### 1. 使用项目基础设施
- **必须使用 LoggerModule**：在所有 Service 中注入 `HLogger` 进行日志记录
- **必须使用 ConfigModule**：通过 `ConfigService` 读取配置，不要硬编码
- **必须使用权限装饰器**：合理使用 `@AuthRoles` 控制接口权限
- **必须使用 API 版本控制**：所有 Controller 都要指定 `version: [VERSION_NEUTRAL, '1']`

### 2. 依赖注入
- 优先使用依赖注入而非直接实例化
- 通过构造函数或属性注入（`@Inject`）依赖
- Logger 使用属性注入：`@Inject(HLOGGER_TOKEN) private logger: HLogger`

### 3. 异步处理
- **必须使用 `async/await`** 处理异步操作，**禁止混用 `await` 和 `.then()`**
- 使用 `try-catch` 进行错误处理，不使用 `.catch()`
- 为 Promise 提供明确的类型

```typescript
// ✅ 正确：使用 async/await + try-catch
async function fetchData() {
  try {
    const response = await api.getData();
    const data = response.data;
    this.log('获取数据成功');
    return data;
  } catch (error) {
    this.error(`获取数据失败: ${error.message}`);
    throw new BusinessException('获取数据失败');
  }
}

// ❌ 错误：混用 await 和 .then()
async function fetchData() {
  const data = await api.getData()
    .then(res => res.data)
    .catch(e => console.error(e));
  return data;
}
```

### 4. 环境变量
- 使用 `ConfigService` 管理配置（已配置为全局）
- 创建配置类型定义
- 不要在代码中硬编码敏感信息
- 支持多环境配置文件

### 5. 数据库查询
- 使用 TypeORM Query Builder 处理复杂查询
- 使用事务处理关联操作
- 注意 N+1 查询问题，合理使用 relations
- 新模块必须在 Module 中注册实体：`TypeOrmModule.forFeature([Entity])`

### 6. API 设计
- 遵循 RESTful 规范
- 使用合适的 HTTP 状态码
- 提供清晰的错误信息（使用 BusinessException）
- 合理设置接口权限（user / admin / public）

### 7. 性能优化
- 使用分页处理大量数据（nestjs-typeorm-paginate）
- 合理使用缓存
- 数据库查询优化（索引、查询优化）

## 开发流程

### 1. 功能开发
1. **创建功能分支**
2. **定义 Entity 和 DTO**
3. **实现 Service 层业务逻辑**
   - 必须注入并使用 `HLogger`
   - 使用 `ConfigService` 读取配置
   - 使用 `BusinessException` 处理异常
4. **创建 Controller 暴露 API**
   - 指定 API 版本
   - 使用 `@AuthRoles` 设置权限
   - 需要用户信息时使用 `@UserParams()`
5. **在 Module 中注册**
   - 注册 Controller 和 Provider
   - 使用 `TypeOrmModule.forFeature([Entity])` 注册实体
6. **在 AppModule 中导入新模块**

### 2. 模块创建示例
```typescript
// post.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { PostController } from './post.controller';
import { PostService } from './post.service';
import { PostEntity } from '@/entities';

@Module({
  imports: [TypeOrmModule.forFeature([PostEntity])],
  controllers: [PostController],
  providers: [PostService],
  exports: [PostService], // 如果其他模块需要使用
})
export class PostModule {}
```

```typescript
// app.module.ts - 导入新模块
import { PostModule } from '@/module/post';

@Module({
  imports: [
    ConfigModule.forRoot({ ... }),
    TypeOrmModule.forRootAsync({ ... }),
    LoggerModule,
    UserModule,
    PostModule, // 添加新模块
    EventEmitterModule.forRoot(),
  ],
  // ...
})
export class AppModule {}
```

### 3. 代码审查要点
- **TypeScript 类型完整性**：是否有明确的类型声明，是否避免使用 `any`
- **基础设施使用**：是否使用了 Logger 和 ConfigService
- **权限控制**：`@AuthRoles` 使用是否正确
- **错误处理**：是否使用 `BusinessException` 处理业务异常
- **函数长度**：单函数是否超过 150 行，是否需要拆分
- **文档注释**：是否添加了 JSDoc 注释，参数、返回值、逻辑说明是否完整
- **命名规范**：变量、常量、接口命名是否符合规范
- **代码可读性**：是否有必要的行内注释，逻辑是否清晰
- **性能问题**：是否有潜在的性能问题（N+1 查询、重复计算等）
- **安全性**：是否有输入验证，是否有安全隐患

### 4. 提交前检查
- 运行 `pnpm --filter @applog/backend run lint` 检查代码
- 运行 `pnpm --filter @applog/backend run format` 格式化代码
- 确保类型检查通过
- 测试 API 接口
- 使用 `pnpm c` 规范提交信息

## 注意事项

### 项目基础设施使用（重要）
- **日志记录**: 
  - **必须使用** `HLogger`（来自 `@reus-able/nestjs`）
  - 在所有 Service 中通过 `@Inject(HLOGGER_TOKEN)` 注入
  - 记录重要操作和错误信息
  
- **配置管理**:
  - **必须使用** `ConfigService` 读取配置
  - 不要硬编码任何配置值（URL、端口、密钥等）
  
- **权限控制**:
  - 使用 `@AuthRoles('user')` 或 `@AuthRoles('admin')` 装饰器
  - 不需要权限的公开接口不使用装饰器
  - 需要当前用户信息时使用 `@UserParams()` 装饰器
  
- **API 版本控制**:
  - 所有 Controller 必须指定版本：`version: [VERSION_NEUTRAL, '1']`
  
- **异常处理**:
  - 业务异常优先使用 `BusinessException`
  - 全局异常过滤器会自动统一响应格式

### 安全性
- **输入验证**: 
  - 使用 `class-validator` 装饰器验证所有用户输入
  - 全局 ValidationPipe 已配置自动类型转换
    
- **数据安全**:
  - 使用参数化查询防止 SQL 注入（TypeORM 已支持）
  - 敏感数据加密存储
    
- **认证授权**:
  - 全局 AuthGuard 已配置
  - 合理使用 `@AuthRoles` 控制访问权限

---
> Source: [youranreus/Applog](https://github.com/youranreus/Applog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
