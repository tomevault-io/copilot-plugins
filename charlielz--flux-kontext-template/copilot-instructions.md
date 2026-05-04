## flux-kontext-template

> 穷尽一切方法思考，chat上下文中文Always respond in 中文，对话框讨论永远用Chinese回复我，代码中只能用中文注释但是网页文案不能有中文；永远只使用3000端口不新增端口运行，3000端口一直在启动所以不需要新增服务器启动，你可以查看所有文件来检查并修改直到项目不再出现错误，每次项目更新和改动也都写到readme.md文档里面哈，每个步骤都一定要上传到GIT的默认MASTER分支并写好COMMIT MESSAGES以放在代码改崩方便备份恢复，不要直接问我要很多规则和答案，而是帮我规划好不同方案，一步步排查所有可能问题，但是绝不能擅自轻易删除任何文件，删改文件必须在跟我确认之后再去执行

# Cursor Rules for Next.js 15 + TypeScript Project


 穷尽一切方法思考，chat上下文中文Always respond in 中文，对话框讨论永远用Chinese回复我，代码中只能用中文注释但是网页文案不能有中文；永远只使用3000端口不新增端口运行，3000端口一直在启动所以不需要新增服务器启动，你可以查看所有文件来检查并修改直到项目不再出现错误，每次项目更新和改动也都写到readme.md文档里面哈，每个步骤都一定要上传到GIT的默认MASTER分支并写好COMMIT MESSAGES以放在代码改崩方便备份恢复，不要直接问我要很多规则和答案，而是帮我规划好不同方案，一步步排查所有可能问题，但是绝不能擅自轻易删除任何文件，删改文件必须在跟我确认之后再去执行
，遇到需要我提供信息的一定要询问我，挨个给我解释代码文件和逻辑结构，不能基于任何假设来做代码修改, 必须查看代码文件结构去确认实际情况, 最后再去执行所有代码修改的动作，一旦出现解决不了的复杂项目就打印函数和程序调用链自己来分析：修复三律：1️⃣ 精：复杂度≤原方案80%，2️⃣ 准：直击根本原因 ，3️⃣ 净：0技术债务(SonarQube✔️)，⚙️ 三步走：① 溯源 → 函数，方法调用链→ 错误触发路径  ② 拆解 → 给出3个SOLID++方案，不能基于任何假设前提条件，必须要跟我确认具体的API和项目文档，不需要重启开发服务器，一直保持3000端口打开即可，像小白一样教导我所有基础理论知识， 绝不能擅自轻易删除任何文件，删改文件必须在跟我确认之后再去执行，先仔细阅读所有项目文件查看相关代码搞清楚所有情况和错误问题本质原因和可能解决方案再跟我确认后去动手删改代码哈

注意错误：添加 "use client" 指令时，不能同时导出 metadata，因为 metadata 只能在服务器组件中使用


  You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.
  
  Code Style and Structure
  - Write concise, technical TypeScript code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  
  Naming Conventions
  - Use lowercase with dashes for directories (e.g., components/auth-wizard).
  - Favor named exports for components.
  
  TypeScript Usage
  - Use TypeScript for all code; prefer interfaces over types.
  - Avoid enums; use maps instead.
  - Use functional components with TypeScript interfaces.
  
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative JSX.
  
  UI and Styling
  - Use Shadcn UI, Radix, and Tailwind for components and styling.
  - Implement responsive design with Tailwind CSS; use a mobile-first approach.
  
  Performance Optimization
  - Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
  - Wrap client components in Suspense with fallback.
  - Use dynamic loading for non-critical components.
  - Optimize images: use WebP format, include size data, implement lazy loading.
  
  Key Conventions
  - Use 'nuqs' for URL search parameter state management.
  - Optimize Web Vitals (LCP, CLS, FID).
  - Limit 'use client':
    - Favor server components and Next.js SSR.
    - Use only for Web API access in small components.
    - Avoid for data fetching or state management.
  
  Follow Next.js docs for Data Fetching, Rendering, and Routing.    
  

## 🚨 Critical Next.js Rules

### 1. "use client" vs metadata 冲突规则
- ❌ **绝对禁止**: 在同一个文件中同时使用 `"use client"` 和 `export const metadata`
- ✅ **正确做法**: 
  - 服务器组件: 只处理 metadata，不使用 "use client"
  - 客户端组件: 只处理交互逻辑，使用 "use client"
- 🔧 **解决方案**: 页面分离架构
  ```typescript
  // ✅ 服务器组件 (page.tsx)
  import type { Metadata } from 'next'
  import { PageContent } from '@/components/PageContent'
  
  export const metadata: Metadata = { /* SEO配置 */ }
  export default function Page() {
    return <PageContent />
  }
  
  // ✅ 客户端组件 (PageContent.tsx)
  "use client"
  import { useState } from "react"
  export function PageContent() {
    // 所有交互逻辑
  }
  ```

### 2. Next.js App Router 架构规则
- ✅ **layout.tsx**: 只能是服务器组件，处理全局配置
- ✅ **page.tsx**: 优先使用服务器组件，需要交互时分离为客户端组件
- ✅ **loading.tsx**: 服务器组件，处理加载状态
- ✅ **error.tsx**: 必须是客户端组件 ("use client")
- ✅ **not-found.tsx**: 服务器组件

### 3. 组件命名和导出规则
- ✅ **页面组件**: 使用 default export
- ✅ **可复用组件**: 使用 named export
- ✅ **客户端组件**: 文件名以 Content.tsx 结尾 (如 DashboardContent.tsx)
- ✅ **服务器组件**: 直接使用 page.tsx 或组件名.tsx

### 4. TypeScript 和导入规则
- ✅ **类型导入**: 使用 `import type { Metadata } from 'next'`
- ✅ **组件导入**: 使用 `import { Component } from '@/components/Component'`
- ✅ **避免循环依赖**: 检查导入路径，避免 A→B→A 的循环

### 5. 状态管理规则
- ✅ **useState/useEffect**: 只能在客户端组件中使用
- ✅ **usePathname/useRouter**: 只能在客户端组件中使用
- ✅ **服务器组件**: 只能使用 async/await 和服务器端 API

## 🎯 常见错误和解决方案

### 错误1: Event handlers in Server Components
```typescript
// ❌ 错误
export default function Page() {
  return <button onClick={() => {}}>Click</button>
}

// ✅ 正确
"use client"
export function PageContent() {
  return <button onClick={() => {}}>Click</button>
}
```

### 错误2: Hooks in Server Components
```typescript
// ❌ 错误
export default function Page() {
  const [state, setState] = useState(false)
  return <div>{state}</div>
}

// ✅ 正确
"use client"
export function PageContent() {
  const [state, setState] = useState(false)
  return <div>{state}</div>
}
```

### 错误3: Script onLoad in Server Components
```typescript
// ❌ 错误
<Script src="..." onLoad={() => {}} />

// ✅ 正确
<Script src="..." strategy="lazyOnload" />
```

## 🔧 代码质量规则

### 1. 中文注释规则
- ✅ **代码注释**: 必须使用中文
- ✅ **网页文案**: 必须使用英文
- ✅ **变量名**: 使用英文
- ✅ **函数名**: 使用英文

### 2. 端口和服务器规则
- ✅ **固定端口**: 永远只使用 3000 端口
- ✅ **不新增端口**: 不创建新的服务器端口
- ✅ **保持运行**: 3000 端口一直保持启动状态

### 3. Git 提交规则
- ✅ **每次更新**: 必须提交到 Git 默认 master 分支
- ✅ **Commit 格式**: 使用中文描述具体改动
- ✅ **更新文档**: 每次项目更新都要更新 README.md

## 🚀 性能优化规则

### 1. 组件优化
- ✅ **动态导入**: 大型组件使用 `dynamic(() => import())`
- ✅ **Suspense**: 客户端组件包装在 Suspense 中
- ✅ **避免过度使用**: 最小化 "use client" 的使用

### 2. 图片优化
- ✅ **Next.js Image**: 使用 `next/image` 组件
- ✅ **WebP格式**: 优先使用 WebP 格式
- ✅ **懒加载**: 启用图片懒加载

## 📝 文档更新规则

### 1. README.md 更新
- ✅ **功能变更**: 每次功能更新都要记录
- ✅ **技术债务**: 记录已知问题和解决方案
- ✅ **版本信息**: 更新版本号和更新日志

### 2. 错误处理
- ✅ **三律原则**: 精(复杂度≤80%)、准(直击根因)、净(0技术债务)
- ✅ **三步走**: 溯源→拆解→执行
- ✅ **不基于假设**: 必须查看实际代码文件确认情况 

---
> Source: [CharlieLZ/flux-kontext-template](https://github.com/CharlieLZ/flux-kontext-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
