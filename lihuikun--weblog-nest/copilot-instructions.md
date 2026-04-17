## weblog-nest

> ├── [module-name].controller.ts    # 控制器

# NestJS WebLog 项目开发规范

## 项目架构规范

### 模块目录结构
每个功能模块应遵循以下目录结构：
```
src/[module-name]/
├── [module-name].controller.ts    # 控制器
├── [module-name].service.ts       # 服务层
├── [module-name].module.ts        # 模块定义
├── [module-name].controller.spec.ts # 控制器测试
├── [module-name].service.spec.ts  # 服务测试
├── dto/                           # 数据传输对象
│   ├── create-[module-name].dto.ts
│   └── update-[module-name].dto.ts
└── entities/                      # 实体定义
    └── [module-name].entity.ts
```

## 代码风格规范

### 1. Import 语句规范
- 使用多行导入格式，每个导入一行
- 按以下顺序排列：
  1. NestJS 核心模块
  2. 第三方库
  3. 项目内部模块（相对路径）

```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Delete,
  Put,
  Query,
  ParseIntPipe,
} from '@nestjs/common';
import { ApiBody, ApiOperation, ApiParam } from '@nestjs/swagger';
import { ArticleService } from './article.service';
import { CreateArticleDto } from './dto/create-article.dto';
```

### 2. 控制器规范
- 使用 `@Controller('[resource-name]s')` 命名（复数形式）
- 构造函数注入依赖
- 所有公共方法添加 `@ApiOperation` 装饰器
- 使用适当的HTTP方法：
  - GET: 查询操作
  - POST: 创建操作
  - PUT: 完整更新操作
  - PATCH: 部分更新操作
  - DELETE: 删除操作

```typescript
@Controller('articles')
export class ArticleController {
  constructor(private readonly articleService: ArticleService) {}

  @Get()
  @ApiOperation({ summary: '获取文章列表' })
  async getArticles(
    @Query('page') page: number = 1,
    @Query('pageSize') pageSize: number = 10,
  ) {
    return this.articleService.findAll(page, pageSize);
  }
}
```

### 3. 服务层规范
- 使用 `@Injectable()` 装饰器
- 构造函数注入 Repository 和其他依赖
- 方法命名使用完整的动词：`create`, `findAll`, `findOne`, `update`, `delete`
- 异步方法使用 `async/await`
- 错误处理使用抛出异常而不是返回错误对象

```typescript
@Injectable()
export class ArticleService {
  constructor(
    @InjectRepository(Article)
    private readonly articleRepository: Repository<Article>,
  ) {}

  async create(createArticleDto: CreateArticleDto): Promise<Article> {
    const article = this.articleRepository.create(createArticleDto);
    return await this.articleRepository.save(article);
  }
}
```

### 4. DTO 规范
- 使用 `class-validator` 装饰器进行验证
- 所有属性添加 `@ApiProperty` 装饰器
- 提供有意义的示例值和描述
- 可选属性使用 `@IsOptional()` 和 `?` 标记

```typescript
export class CreateArticleDto {
  @ApiProperty({ description: '文章标题', example: '如何学习 NestJS' })
  @IsNotEmpty()
  @IsString()
  @Length(1, 255)
  title: string;

  @ApiProperty({
    description: '文章封面图 URL',
    example: 'https://example.com/image.jpg',
    required: false,
  })
  @IsOptional()
  @IsString()
  coverImage?: string;
}
```

### 5. 实体规范
- 使用 TypeORM 装饰器
- 主键统一使用 `@PrimaryGeneratedColumn()`
- 时间字段使用 `@CreateDateColumn()` 和 `@UpdateDateColumn()`
- 关联关系明确指定 cascade 选项
- 字段类型明确指定长度和默认值

```typescript
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'varchar', length: 255 })
  title?: string;

  @Column({ type: 'int', default: 0 })
  viewCount?: number;

  @OneToMany(() => Comment, (comment) => comment.article, { cascade: true })
  comments?: Comment[];

  @CreateDateColumn({ type: 'timestamp' })
  createTime?: Date;

  @UpdateDateColumn({ type: 'timestamp' })
  updatedTime?: Date;
}
```

### 6. 模块规范
- 导入顺序：NestJS 模块 → 第三方库 → 项目内部模块
- 在 `imports` 中使用 `TypeOrmModule.forFeature()` 注册实体
- 明确声明 `controllers`, `providers`, 必要时添加 `exports`

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([Article, Comment, Like])],
  controllers: [ArticleController],
  providers: [ArticleService],
  exports: [ArticleService], // 如果需要被其他模块使用
})
export class ArticleModule {}
```

## 权限和认证规范

### 使用装饰器进行权限控制
```typescript
// 普通认证
@UseGuards(AuthGuard)

// 角色权限验证
@RequireRole(Role.ADMIN)

// 获取当前用户ID
async method(@CurrentUserId() userId: number) {}
```

## 数据丰富规范

### 批量获取用户信息
```typescript
// 在Service中注入UserService
constructor(
  private readonly userService: UserService,
) {}

// 使用批量获取方法避免N+1查询
private async enrichWithUserInfo(items: any[]) {
  const userIds = [...new Set(items.map(item => item.senderId))];
  const usersInfo = await this.userService.getUsersBasicInfo(userIds);
  
  return items.map(item => ({
    ...item,
    sender: usersInfo[item.senderId] || { id: item.senderId, nickname: '未知用户' }
  }));
}
```

## 命名规范

### 文件命名
- 模块文件：`[module-name].[type].ts`
- 实体文件：`[entity-name].entity.ts`
- DTO文件：`[action]-[entity-name].dto.ts`
- 测试文件：`[module-name].[type].spec.ts`

### 变量和方法命名
- 使用 camelCase
- 变量名具有描述性
- 方法名使用动词开头
- 常量使用 UPPER_SNAKE_CASE

### 接口和类型命名
- 接口使用 PascalCase
- 泛型使用单个大写字母（T, K, V等）

## API设计规范

### RESTful API 设计
- 资源使用复数形式：`/articles`, `/users`
- 使用HTTP状态码表示操作结果
- 统一的响应格式

### 路由命名
```typescript
GET    /articles           # 获取列表
GET    /articles/:id       # 获取单个资源
POST   /articles           # 创建资源
PUT    /articles/:id       # 完整更新
PATCH  /articles/:id       # 部分更新
DELETE /articles/:id       # 删除资源

# 管理后台接口
GET    /articles/all       # 管理员获取所有资源
DELETE /articles/:id       # 管理员删除（需要权限验证）
```

## 数据库设计规范

### 字段命名
- 主键：`id`
- 时间字段：`createTime`, `updatedTime`
- 外键：`[entity]Id`（如：`categoryId`, `userId`）
- 计数字段：`[name]Count`（如：`viewCount`, `likeCount`）

### 字段类型
- 字符串：明确指定长度 `varchar(255)`
- 文本：使用 `text` 类型
- 计数：使用 `int` 并设置默认值 0
- 时间：使用 `timestamp` 类型

## 错误处理规范

### 使用NestJS内置异常
```typescript
// 400 Bad Request
throw new BadRequestException('参数错误');

// 401 Unauthorized  
throw new UnauthorizedException('未认证');

// 403 Forbidden
throw new ForbiddenException('权限不足');

// 404 Not Found
throw new NotFoundException('资源不存在');
```

## 测试规范

### 单元测试
- 为每个Service和Controller编写测试
- 测试文件使用 `.spec.ts` 后缀
- 使用描述性的测试用例名称

## 代码质量规范

### 注释规范
- 为复杂业务逻辑添加注释
- API文档使用Swagger装饰器
- 类和方法使用JSDoc注释

### 性能优化
- 避免N+1查询问题
- 使用批量查询
- 合理使用数据库索引
- 实现数据分页

### 类型安全
- 启用TypeScript严格模式
- 避免使用 `any` 类型
- 为所有公共API定义明确的类型 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lihuikun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
