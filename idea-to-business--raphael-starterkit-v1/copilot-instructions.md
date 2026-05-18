## raphael-starter

> 这是一个使用 Supabase 和 Creem 构建的 Next.js 启动套件，具有身份验证、服务器端渲染、主题支持、订阅管理和支付功能。

# 项目背景
这是一个使用 Supabase 和 Creem 构建的 Next.js 启动套件，具有身份验证、服务器端渲染、主题支持、订阅管理和支付功能。

# 环境配置
项目使用以下环境变量（.env）：
- `NEXT_PUBLIC_SUPABASE_URL`: Supabase 项目 URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase 匿名密钥
- `SUPABASE_SERVICE_ROLE_KEY`: Supabase 服务角色密钥
- `CREEM_WEBHOOK_SECRET`: Creem webhook 签名验证密钥
- `CREEM_API_KEY`: Creem API 密钥
- `CREEM_API_URL`: Creem API URL
- `NEXT_PUBLIC_SITE_URL`: 站点 URL（用于认证回调）
- `CREEM_SUCCESS_URL`: 支付成功后的重定向 URL

# 编码标准
- 使用 TypeScript
- 使用函数组件和 Hooks，避免类组件
- 变量和函数名使用 camelCase 规范，组件名使用 PascalCase
- 使用 Tailwind CSS 进行样式设计，遵循 shadcn/ui 组件模式
- 保持深色/浅色主题兼容性，使用主题提供者进行动态主题设置

# 文件结构
- `app/`：Next.js App Router 页面和布局
  - `(auth-pages)/`：与认证相关的页面
  - `auth/`：认证回调处理
  - `dashboard/`：受保护的仪表板页面
  - `api/webhooks/creem/`：Creem webhook 处理路由
- `components/`：可复用的 React 组件
- `utils/`：实用函数和辅助工具
  - `supabase/`：Supabase 客户端配置和订阅管理
  - `creem/`：Creem 相关工具函数
- `lib/`：库代码和共享实用工具
- `public/`：静态资源
- `types/`：TypeScript 类型
- `hooks/`：自定义钩子
  - `use-toast.ts`：提示通知
  - `use-user.ts`：用户钩子
  - `use-subscription.ts`：订阅钩子

# 身份验证
- 使用服务器操作进行身份验证操作
- 通过适当的重定向处理错误
- 在受保护的路由中检查身份验证状态
- 使用中间件进行路由保护

# 订阅和支付系统
- 使用 Creem 处理支付和订阅
- 支持多种订阅事件处理：
  - checkout.completed：结账完成
  - subscription.active：订阅激活
  - subscription.paid：订阅支付
  - subscription.canceled：订阅取消
  - subscription.expired：订阅过期
  - subscription.trialing：订阅试用
- 实现订阅状态同步
- 支持积分购买系统
- 使用 webhook 处理异步事件

# 服务器操作
- 使用服务器操作提交表单
- 实现适当的错误处理
- 返回适当的重定向
- 验证输入数据

# 错误处理
- 对异步操作使用 try-catch 块
- 实现适当的错误消息
- 使用 FormMessage 组件处理表单错误
- 适当处理身份验证错误
- 处理支付和订阅相关错误

# 常见陷阱避免
- 不要绕过身份验证中间件
- 始终处理身份验证错误
- 在受保护的路由中检查用户会话
- 不要混合不同的样式方法，保持主题一致性
- 不要暴露敏感操作，始终验证输入数据
- 确保正确处理 webhook 签名验证
- 妥善保管所有 API 密钥和敏感信息
- 实现适当的订阅状态同步机制

# 添加新功能时
- 遵循现有的文件结构
- 添加适当的 TypeScript 类型
- 实现适当的错误处理
- 更新相关组件
- 遵循现有的命名约定
- 添加适当的验证
- 保持主题兼容性
- 确保订阅和支付相关功能的安全性

# 测试
- 测试身份验证流程
- 测试受保护的路由
- 测试表单提交
- 测试错误场景
- 测试主题切换
- 测试响应式设计
- 测试支付流程
- 测试订阅状态变更
- 测试 webhook 处理
- 测试积分系统

# 需要帮助？
- 检查类似文件中的现有实现
- 仔细查看错误消息
- 查阅使用的库的文档
- 遵循代码库中已建立的模式
- 参考 Supabase 和 Creem 的官方文档

记住：这个启动套件旨在可维护和可扩展。在进行更改或添加时，始终遵循已建立的模式和最佳实践。特别注意支付和订阅相关功能的安全性，确保正确处理所有敏感信息。

---
> Source: [Idea-To-Business/raphael-starterkit-v1](https://github.com/Idea-To-Business/raphael-starterkit-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
