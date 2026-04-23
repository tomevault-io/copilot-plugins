## theme-anheyu-nova

> 本主题项目是 anheyu-pro 的博客前端主题，基于 Next.js 构建，需要与 anheyu-pro（及其基础版 anheyu-app）后端进行 API 交互。

# AnHeYu Blog Theme API 集成规范

本主题项目是 anheyu-pro 的博客前端主题，基于 Next.js 构建，需要与 anheyu-pro（及其基础版 anheyu-app）后端进行 API 交互。

## 项目关系

```
anheyu-app        # 基础版后端（Go + Gin）
    ↓ 继承扩展
anheyu-pro        # Pro 版后端（扩展了会员、知识库、AI 等功能）
    ↓ 提供 API
anheyu-blog-theme # 本项目 - Next.js 博客主题
```

## API 调用规范

### 1. API 路径前缀

后端 API 遵循以下路径前缀规则：

| 前缀 | 说明 | 认证要求 |
|------|------|----------|
| `/api/public/*` | 公开接口，无需认证 | 无 |
| `/api/pro/*` | Pro 版扩展接口 | 部分需要认证 |
| `/api/pro/public/*` | Pro 版公开接口 | 无 |
| `/api/pro/admin/*` | Pro 版管理接口 | 需要管理员权限 |
| `/api/*` | 基础接口（anheyu-app） | 部分需要认证 |

### 2. API 响应格式

所有 API 返回统一的 JSON 格式：

```typescript
interface ApiResponse<T> {
  code: number;      // 状态码：200 = 成功
  message: string;   // 状态消息
  data: T;           // 响应数据
}
```

### 3. 常用 API 端点

#### 文章相关（基础版 + Pro 版共有）

```typescript
// 公开接口 - 无需认证
GET  /api/public/articles              // 获取文章列表
GET  /api/public/articles/home         // 获取首页推荐文章
GET  /api/public/articles/random       // 获取随机文章
GET  /api/public/articles/archives     // 获取文章归档
GET  /api/public/articles/statistics   // 获取文章统计

// Pro 版公开接口
GET  /api/pro/articles/:id/content     // 获取文章详情（支持全文隐藏验证）
GET  /api/pro/public/feed              // 获取混合内容流（文章+商品）

// 分类和标签
GET  /api/post-categories              // 获取分类列表
GET  /api/post-categories/slug/:slug   // 根据 slug 获取分类
GET  /api/post-tags                    // 获取标签列表
GET  /api/post-tags/slug/:slug         // 根据 slug 获取标签
```

#### 评论相关

```typescript
GET  /api/public/comments              // 获取评论列表
GET  /api/public/comments/:id/children // 获取子评论
POST /api/public/comments              // 发表评论
POST /api/public/comments/:id/like     // 点赞评论
GET  /api/public/comments/recent       // 获取最新评论
```

#### 网站配置

```typescript
GET  /api/public/site-config           // 获取网站配置
GET  /api/public/friend-links          // 获取友情链接
POST /api/public/friend-links/apply    // 申请友情链接
```

#### Pro 版专有功能

```typescript
// 知识库问答
POST /api/pro/knowledge/ask            // 知识库问答（非流式）
GET  /api/pro/knowledge/ask/stream     // 知识库问答（流式 SSE）
GET  /api/pro/knowledge/search         // 知识库搜索

// 会员系统
GET  /api/pro/membership/plans         // 获取会员套餐
GET  /api/pro/membership/my            // 获取我的会员信息
GET  /api/pro/membership/access/check  // 检查会员访问权限

// 全文隐藏验证
GET  /api/pro/articles/:id/full-text-hidden-config  // 获取全文隐藏配置
POST /api/pro/articles/:id/full-text-verify         // 验证密码获取全文

// 文档系列
GET  /api/public/doc-series            // 获取文档系列列表
GET  /api/public/doc-series/:id        // 获取文档系列详情
GET  /api/public/doc-series/:id/articles // 获取系列下的文章
```

### 4. 本项目 API 客户端使用

本项目已封装 API 客户端在 `src/lib/api/` 目录：

```typescript
// 引入 API 服务
import { articlesApi, categoriesApi, tagsApi, commentsApi, siteApi } from '@/lib/api';

// 使用示例
const articles = await articlesApi.getList({ page: 1, pageSize: 10 });
const article = await articlesApi.getDetail(id);
const categories = await categoriesApi.getList();
```

### 5. 添加新 API 的步骤

1. **定义类型** - 在 `src/lib/api/types.ts` 添加接口类型
2. **实现服务** - 在 `src/lib/api/services.ts` 添加 API 调用函数
3. **创建 Hook** - 在 `src/hooks/` 创建对应的 React Hook（可选）
4. **更新 Store** - 如需状态管理，在 `src/store/` 添加 Zustand store

### 6. 环境变量配置

```env
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:8080  # 后端 API 地址
```

## 注意事项

### Pro 版 vs 基础版 API 差异

| 功能 | 基础版 (anheyu-app) | Pro 版 (anheyu-pro) |
|------|---------------------|---------------------|
| 文章详情 | `/api/public/articles/:id` | `/api/pro/articles/:id/content` |
| 文章创建 | `/api/articles` | `/api/pro/articles` |
| 标签创建 | `/api/post-tags` | `/api/pro/post-tags` |
| 分类创建 | `/api/post-categories` | `/api/pro/post-categories` |
| 会员功能 | ❌ | ✅ `/api/pro/membership/*` |
| 知识库 | ❌ | ✅ `/api/pro/knowledge/*` |
| 全文隐藏 | ❌ | ✅ `/api/pro/articles/*/full-text-*` |
| AI 摘要 | ❌ | ✅ `/api/pro/article/ai-summary` |
| 商品功能 | ❌ | ✅ `/api/pro/products/*` |

### 文章内容字段

Pro 版文章包含额外字段：

```typescript
interface Article {
  // ... 基础字段
  
  // Pro 版特有字段
  has_paid_content?: boolean;      // 是否有付费内容
  has_password_content?: boolean;  // 是否有密码保护内容
  has_full_text_hidden?: boolean;  // 是否开启全文隐藏
  toc?: TocItem[];                 // 文章目录
}
```

### SSE 流式响应处理

对于知识库问答等流式接口，使用 EventSource：

```typescript
const eventSource = new EventSource('/api/pro/knowledge/ask/stream?question=' + encodeURIComponent(question));

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // 处理流式数据
};

eventSource.onerror = () => {
  eventSource.close();
};
```

## 开发调试

### 本地开发时代理配置

在 `next.config.ts` 中配置代理：

```typescript
async rewrites() {
  return [
    {
      source: '/api/:path*',
      destination: 'http://localhost:8080/api/:path*', // 后端地址
    },
  ];
}
```

### 参考后端项目

开发新功能时，应参考后端 API 实现：
- **anheyu-app**: `anheyu-app/assets/src/api/` - 基础 API
- **anheyu-pro**: `anheyu-pro/assets/src/api/` - Pro 版扩展 API

后端 API 文件结构：
```
api/
├── post/           # 文章相关
├── comment/        # 评论相关
├── knowledge.ts    # 知识库 (Pro)
├── membership.ts   # 会员 (Pro)
├── payment.ts      # 支付 (Pro)
├── site.ts         # 网站配置
└── ...
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anzhiyu-c) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
