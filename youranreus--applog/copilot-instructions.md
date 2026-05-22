## base

> 这是一个使用 TypeScript 开发的全栈博客应用，采用 pnpm workspace 进行 monorepo 管理。


# AppLog Project - 通用开发规范

## 项目概述
这是一个使用 TypeScript 开发的全栈博客应用，采用 pnpm workspace 进行 monorepo 管理。

## 项目结构
```
applog/
├── packages/
│   ├── backend/        # NestJS 后端服务
│   ├── frontend/       # Vue3 前端应用
│   └── finder/         # 其他模块（待开发）
├── pnpm-workspace.yaml
└── package.json
```

## TypeScript 通用规范

### 通用规则
1. **严格类型检查**: 必须为所有函数、变量、参数提供明确的类型声明
2. **避免 `any`**: 尽可能避免使用 `any`，使用具体类型或泛型
3. **类型推断**: 充分利用 TypeScript 的类型推断能力，但在接口边界必须显式声明
4. **类型导入**: 使用 `import type` 导入仅用于类型的导入

## 命名规范

### 文件命名
- **实体、DTO、类文件**: PascalCase（如 `User.ts`, `CreateUserDto.ts`）
- **工具函数、模块**: kebab-case（如 `user-utils.ts`, `user.controller.ts`）

### 代码命名
- **类名**: PascalCase（如 `UserService`, `PostEntity`）
- **变量、字段**: camelCase（如 `userName`, `userId`, `createdAt`）
- **函数、方法**: camelCase（如 `findOne`, `createUser`, `getUserList`）
- **常量**: UPPER_SNAKE_CASE（如 `MAX_COUNT`, `DEFAULT_PAGE_SIZE`, `API_VERSION`）
- **Interface 接口**: **必须使用 I 前缀 + PascalCase**（如 `IUser`, `IUserResponse`, `ICreateUserDto`）
- **Type 类型别名**: PascalCase（如 `UserRole`, `PostStatus`）

### 类型声明规范
```typescript
// ✅ 正确：Interface 使用 I 前缀
interface IUser {
  id: string;
  userName: string;  // 字段使用 camelCase
  email: string;
}

// ✅ 正确：Type 类型别名使用 PascalCase
type UserRole = 'admin' | 'user';
type PostStatus = 'draft' | 'published' | 'archived';

// ✅ 正确：常量使用 UPPER_SNAKE_CASE
const MAX_PAGE_SIZE = 100;
const DEFAULT_TIMEOUT = 3000;

// ❌ 错误：Interface 不使用 I 前缀
interface User {  // 应该是 IUser
  id: string;
}

// ❌ 错误：字段使用 snake_case
interface IUser {
  user_name: string;  // 应该是 userName
  created_at: Date;   // 应该是 createdAt
}
```

### 枚举使用规范
**尽量避免使用 enum 枚举**，推荐使用以下替代方案：

```typescript
// ❌ 不推荐：使用 enum
enum UserRole {
  Admin = 'admin',
  User = 'user',
}

// ✅ 推荐：使用 type + const 对象
type UserRole = 'admin' | 'user';
const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user',
} as const;

// ✅ 推荐：使用 type + union
type PostStatus = 'draft' | 'published' | 'archived';

// ✅ 推荐：需要常量时使用 const 对象
const POST_STATUS = {
  DRAFT: 'draft',
  PUBLISHED: 'published',
  ARCHIVED: 'archived',
} as const;

type PostStatus = typeof POST_STATUS[keyof typeof POST_STATUS];
```

**枚举替代方案的优势**：
- 更好的类型推断
- 更灵活的类型操作
- 避免 enum 的运行时开销
- 更符合 TypeScript 最佳实践

## 代码质量与可维护性

### 函数长度控制
- **单函数行数不能超过 150 行**
- 超过限制时必须拆分为多个小函数
- 保持函数职责单一，易于测试和维护

### 文档注释（必须添加）
- **所有函数必须添加 JSDoc 注释**
- 必须标注参数（`@param`）和返回值（`@returns`）
- 必须简明标注函数逻辑和主要流程
- 复杂逻辑需要在函数内添加行内注释说明

#### JSDoc 注释格式
```typescript
/**
 * 函数功能简短描述（一句话说明功能）
 * @param paramName - 参数说明
 * @param anotherParam - 另一个参数说明
 * @returns 返回值说明
 * @throws {ExceptionType} 可能抛出的异常说明
 * 
 * 逻辑说明：
 * 1. 第一步操作
 * 2. 第二步操作
 * 3. 第三步操作
 */
async functionName(paramName: string, anotherParam: number): Promise<ReturnType> {
  // 关键步骤注释
  const result = await someOperation();
  
  // 条件判断说明
  if (condition) {
    // 分支逻辑说明
    return result;
  }
  
  return defaultValue;
}
```

#### 注释要点
1. **简洁明了**：避免冗长描述，突出重点
2. **说明逻辑**：重点说明"为什么"而不只是"做什么"
3. **标注异常**：明确可能抛出的异常类型和原因
4. **更新及时**：代码变更时同步更新注释
5. **中文优先**：项目使用中文注释，提高团队理解效率

### 代码质量要求
1. **禁止使用 `any`**: 除非有充分理由，否则禁止使用 `any` 类型
2. **类型安全**: 所有 API 调用、数据库操作都应有类型保护
3. **错误处理**: 必须妥善处理所有可能的错误情况
4. **代码复用**: 提取公共逻辑到服务或工具函数
5. **异步处理**: **必须使用 `async/await`** 处理异步操作，**禁止混用 `await` 和 `.then()`**

```typescript
// ✅ 正确：使用 async/await + try-catch
async function fetchData() {
  try {
    const response = await api.getData();
    const data = response.data;
    return data;
  } catch (error) {
    throw new Error(`获取数据失败: ${error.message}`);
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

## pnpm Workspace 使用

### 包命名
- 统一使用 `@applog/` 前缀
- 示例: `@applog/backend`, `@applog/frontend`, `@applog/finder`

### 常用命令
```bash
# 安装依赖
pnpm install

# 为特定包添加依赖
pnpm --filter @applog/backend add <package>

# 运行开发服务器
pnpm fe          # 前端
pnpm be          # 后端
pnpm dev         # 全部

# 构建
pnpm build:fe    # 构建前端
pnpm build:be    # 构建后端
pnpm build       # 构建全部
```

### 跨包引用
- 在 package.json 中添加工作区依赖: `"@applog/shared": "workspace:*"`
- 确保被依赖的包正确导出类型

## Git 提交规范（约定式提交）

### 重要说明
**当 AI Agent 帮助创建代码提交时，必须严格遵守以下约定式提交规范（Conventional Commits）。**

### 使用 Commitizen
```bash
pnpm c  # 使用交互式提交工具
```

### Commit Message 格式
```
<type>(<scope>): <subject>

[可选的 body]

[可选的 footer]
```

#### 格式说明
- **type**: 提交类型（必需）
- **scope**: 影响范围（可选，建议添加）
- **subject**: 简短描述（必需，不超过 50 字符）
- **body**: 详细描述（可选，说明改动的原因和内容）
- **footer**: 备注信息（可选，如关联的 issue、破坏性变更说明）

### Type 类型（必须使用）

#### 主要类型
- **feat**: 新功能（feature）
  - 添加新的功能特性
  - 示例：`feat(user): 添加用户头像上传功能`

- **fix**: 修复 Bug
  - 修复代码中的错误
  - 示例：`fix(post): 修复文章列表分页错误`

- **docs**: 文档更新
  - 仅修改文档（README、注释等）
  - 示例：`docs(readme): 更新项目安装说明`

- **style**: 代码格式调整
  - 不影响代码功能的格式修改（空格、缩进、分号等）
  - 示例：`style(backend): 统一代码缩进为 2 空格`

- **refactor**: 代码重构
  - 既不是新功能也不是 bug 修复的代码改动
  - 示例：`refactor(user): 优化用户服务层代码结构`

- **perf**: 性能优化
  - 提升性能的代码改动
  - 示例：`perf(post): 优化文章列表查询性能`

- **test**: 测试相关
  - 添加或修改测试代码
  - 示例：`test(user): 添加用户注册单元测试`

- **chore**: 构建/工具链更新
  - 构建过程或辅助工具的变动
  - 示例：`chore(deps): 升级 nestjs 到 10.3.7`

#### 其他类型
- **build**: 构建系统或外部依赖的变更
  - 示例：`build(npm): 修改 package.json 脚本`

- **ci**: CI 配置文件和脚本的变更
  - 示例：`ci(github): 添加 GitHub Actions 工作流`

- **revert**: 回滚之前的提交
  - 示例：`revert: 回滚 feat(user): 添加用户头像上传功能`

### Scope 范围（建议使用）

#### Backend Scope
- `user`: 用户模块
- `post`: 文章模块
- `comment`: 评论模块
- `auth`: 认证模块
- `config`: 配置相关
- `database`: 数据库相关
- `api`: API 接口
- `middleware`: 中间件
- `guard`: 守卫
- `dto`: 数据传输对象
- `entity`: 实体

#### Frontend Scope
- `assets`: 静态资源
- `components`: 公共组件
- `pages`: 页面视图
- `store`: 状态管理
- `router`: 路由
- `hooks`: 组合式函数
- `utils`: 工具函数
- `types`: 类型定义
- `constants`: 常量
- `api`: API 请求

#### 通用 Scope
- `deps`: 依赖项
- `config`: 配置文件
- `docs`: 文档
- `types`: 类型定义

### Subject 编写规范

#### 要求
1. **使用中文**（项目团队为中文母语）
2. **使用动词开头**：添加、修复、优化、重构等
3. **不超过 50 个字符**
4. **不要使用句号结尾**
5. **言简意赅**：准确描述改动内容

#### ✅ 好的 Subject 示例
```
feat(user): 添加用户登录功能
fix(post): 修复文章删除时评论未级联删除的问题
refactor(auth): 重构 JWT 验证逻辑
perf(post): 优化文章列表查询，添加数据库索引
docs(api): 更新 API 文档，添加用户接口说明
style(backend): 统一使用单引号
chore(deps): 升级 TypeScript 到 5.3.0
```

#### ❌ 不好的 Subject 示例
```
feat: update                          // 太模糊，没有说明更新了什么
fix: bug fix                          // 没有说明修复了什么 bug
feat(user): add user login function.  // 使用了英文，且有句号
feat(user): 添加了用户登录功能，包括密码加密和 JWT 生成  // 太长，超过 50 字符
fix: 修复                             // 没有 scope，描述不清晰
```

### Body 编写规范（可选）

当提交需要更详细的说明时添加 body：

```
feat(post): 添加文章草稿自动保存功能

- 每 30 秒自动保存草稿到数据库
- 用户可以从草稿列表恢复编辑
- 草稿保存不触发文章发布流程
- 添加草稿数量限制，最多保存 10 篇

相关 issue: #123
```

#### Body 规范
1. 与 subject 之间空一行
2. 使用列表说明主要改动点
3. 解释**为什么**做这个改动，而不仅是**做了什么**
4. 每行不超过 72 个字符

### Footer 编写规范（可选）

#### 关联 Issue
```
feat(user): 添加用户权限管理功能

实现了基于角色的权限控制系统

Closes #45
```

#### 破坏性变更（BREAKING CHANGE）
```
feat(api): 重构用户 API 接口

BREAKING CHANGE: 
- 用户 API 路径从 /api/users 改为 /api/v1/users
- 响应数据格式变更，所有接口统一使用 { data, message } 格式
- 删除了 /api/users/list 接口，请使用 /api/v1/users 代替

迁移指南：详见 docs/migration-v1.md
```

### AI Agent 提交代码时的规范

**重要提示：只有当用户明确要求时才能提交代码**

⚠️ **禁止自动提交**：
- ❌ **绝对不要**在用户没有明确要求的情况下提交代码
- ❌ **绝对不要**主动提议"我帮你提交这些改动"
- ✅ **只有当用户明确说**"请提交"、"帮我提交"、"创建提交"等时才能执行 git commit
- ✅ 完成代码改动后，可以提示用户有未提交的改动，但不要主动提交

**当 AI Agent 帮助创建提交时，必须：**

1. ✅ **自动选择正确的 type**：根据代码改动性质选择
2. ✅ **添加合适的 scope**：根据修改的模块选择
3. ✅ **编写清晰的 subject**：使用中文，动词开头，不超过 50 字符
4. ✅ **多个改动时考虑拆分提交**：每个提交只做一件事
5. ✅ **重要改动添加 body**：说明改动原因和影响
6. ✅ **破坏性变更必须标注 BREAKING CHANGE**

#### 提交拆分原则
```bash
# ✅ 好的做法：功能拆分提交
git commit -m "feat(user): 添加用户实体和 DTO"
git commit -m "feat(user): 添加用户服务层逻辑"
git commit -m "feat(user): 添加用户 API 接口"
git commit -m "docs(api): 更新用户接口文档"

# ❌ 不好的做法：所有改动一次提交
git commit -m "feat(user): 完成用户模块开发"
```

### 提交前检查清单

在创建提交前，AI Agent 应该检查：

- [ ] 提交信息格式符合约定式提交规范
- [ ] type 选择正确
- [ ] scope 选择合适（如果适用）
- [ ] subject 使用中文，清晰简洁
- [ ] 多个不相关改动已拆分为多个提交
- [ ] 破坏性变更已标注 BREAKING CHANGE
- [ ] 关联的 issue 已在 footer 中引用

## 最佳实践

### 代码可读性
- 使用有意义的变量名，添加必要的注释
- 避免代码重复，提取公共逻辑，遵循 DRY 原则
- 保持代码结构清晰，逻辑分明

### 测试
- 为关键业务逻辑编写单元测试
- 使用 Jest 进行测试
- 保持测试覆盖率

---
> Source: [youranreus/Applog](https://github.com/youranreus/Applog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
