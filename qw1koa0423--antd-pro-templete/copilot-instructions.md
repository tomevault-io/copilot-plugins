## antd-pro-templete

> - **框架**: React 18 + TypeScript + UMI 4.x

# 开发工作流程与AI协作指南

## 项目基础信息

### 技术栈

- **框架**: React 18 + TypeScript + UMI 4.x
- **UI组件**: Ant Design Pro + Pro Components
- **状态管理**: UMI内置的初始状态管理
- **权限控制**: 基于API权限列表的细粒度权限管理
- **构建工具**: UMI Max + ESBuild
- **包管理**: pnpm

### 项目特色

- 多环境配置支持 (local/alpha/beta/rc/production)
- 基于API权限列表的权限控制系统
- 自定义文件上传组件（支持分片上传）
- 完善的代码规范和类型定义

## API开发规范

### 1. 基于Swagger生成接口

项目使用 `src/services/swaggerApi.json` 作为API规范源：

```typescript
// 标准API响应结构
interface BaseResponse {
  code: number; // 错误码，0成功，其他失败
  msg: string; // 错误信息
  data: any; // 响应数据
}
```

### 2. 服务层文件结构

```
src/services/
├── {module}/           # 按业务模块划分
│   ├── api.ts         # API接口定义
│   └── typings.d.ts   # 类型定义
├── typings.d.ts       # 通用类型定义
└── swaggerApi.json    # Swagger规范文件
```

### 3. API接口开发步骤

1. **分析需求** - 确定业务模块和接口功能
2. **定义类型** - 在对应的typings.d.ts中定义接口类型
3. **实现接口** - 在api.ts中实现具体接口调用
4. **响应处理** - 利用响应拦截器统一处理数据格式
5. **权限控制** - 配置接口权限要求

### 4. 标准API接口模板

```typescript
/**
 * @name 接口名称
 * @description 接口描述
 * @param params 参数类型
 */
export async function apiName(params: ModuleType.RequestParams) {
  return request<ModuleType.ResponseData>(`${API_URL}/api/path`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    data: params,
  });
}
```

## 页面开发规范

### 1. 页面组件结构

```typescript
// 标准页面组件模板
import { PageContainer } from '@ant-design/pro-components';
import { useAccess } from '@umijs/max';

const ModuleNamePage: React.FC = () => {
  const access = useAccess();

  return (
    <PageContainer
      ghost
      header={{
        title: '页面标题',
      }}
    >
      {/* 页面内容 */}
    </PageContainer>
  );
};

export default ModuleNamePage;
```

### 2. 权限控制集成

```typescript
// 组件级权限控制
<Access
  accessible={access.canAccess('specific:api')}
  fallback={<Button disabled>无权限</Button>}
>
  <Button type="primary">有权限操作</Button>
</Access>
```

## AI协作指导原则

### 1. 需求描述最佳实践

当您需要AI帮助开发时，请按以下格式描述需求：

```markdown
## 功能需求

- **模块名称**: 用户管理
- **功能描述**: 实现用户列表查询、新增、编辑、删除功能
- **权限要求**: 需要 user:list, user:create, user:update, user:delete 权限
- **页面路由**: /user/management

## 接口需求

- GET /api/user/list - 获取用户列表
- POST /api/user/create - 创建用户
- PUT /api/user/update - 更新用户
- DELETE /api/user/delete - 删除用户

## UI要求

- 使用ProTable组件展示用户列表
- 支持搜索、分页、排序
- 操作列包含编辑、删除按钮
```

### 2. AI生成代码时应遵循的原则

1. **类型优先**: 始终先定义TypeScript类型，再实现功能
2. **权限集成**: 所有涉及权限的功能都要集成权限控制
3. **组件复用**: 优先使用项目现有的自定义组件
4. **错误处理**: 统一使用项目的错误处理机制
5. **代码规范**: 遵循项目的命名规范和代码风格

### 3. 自动生成内容清单

AI在协助开发时应自动生成：

- [ ] TypeScript类型定义
- [ ] API接口函数
- [ ] 页面组件代码
- [ ] 路由配置
- [ ] 权限配置
- [ ] 测试用例（如需要）

### 4. 常用开发场景

#### 场景1: 新增业务模块

```
我需要创建一个商品管理模块，包含商品列表、新增商品、编辑商品功能
```

AI应该：

1. 创建 `src/services/product/` 目录
2. 定义商品相关的TypeScript类型
3. 实现商品管理的API接口
4. 创建商品管理页面组件
5. 配置相应的路由和权限

#### 场景2: 基于Swagger生成接口

```
根据swagger.json文件中的 /api/order 相关接口，生成订单管理的完整代码
```

AI应该：

1. 解析swagger定义
2. 生成对应的TypeScript类型
3. 创建标准的API接口函数
4. 提供使用示例

#### 场景3: 组件功能增强

```
为CustomUploadButton组件增加图片预览功能
```

AI应该：

1. 分析现有组件结构
2. 保持现有API不变
3. 新增预览相关的props和功能
4. 更新组件文档

## 质量检查清单

每次生成代码后，AI应确保：

- [ ] TypeScript类型定义完整且正确
- [ ] 遵循项目命名规范
- [ ] 集成了必要的权限控制
- [ ] 错误处理机制完善
- [ ] 组件props类型定义清晰
- [ ] 导入导出语句正确
- [ ] 注释规范且完整
- [ ] 代码格式符合项目规范

## 环境和工具

### 开发命令

```bash
pnpm start              # 本地开发
pnpm start:alpha        # Alpha环境
pnpm start:beta         # Beta环境
pnpm run build          # 生产构建
pnpm run lint           # 代码检查
pnpm run type-check     # 类型检查
```

### 推荐的AI辅助工具使用方式

1. 先描述清楚需求和期望的文件结构
2. 让AI生成代码框架和类型定义
3. 逐步完善业务逻辑和UI组件
4. 最后进行权限集成和测试description: globs: alwaysApply: false

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qw1koa0423) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
