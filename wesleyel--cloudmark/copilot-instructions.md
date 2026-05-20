## cloudmark

> "project_description": "Cloudmark 是一个基于云的书签收集工具，允许用户一键保存和组织书签，并可从任何设备访问。核心功能是通过书签小工具(bookmarklet)实现快速保存网页。",

{
  "project_description": "Cloudmark 是一个基于云的书签收集工具，允许用户一键保存和组织书签，并可从任何设备访问。核心功能是通过书签小工具(bookmarklet)实现快速保存网页。",
  
  "architecture": {
    "frontend": "Next.js 15 (App Router)与shadcn UI组件库",
    "styling": "Tailwind CSS，使用shadcn预设的设计系统",
    "backend": "Cloudflare Pages和Workers (Edge运行时)",
    "storage": "Cloudflare KV作为数据存储",
    "deployment": "使用wrangler部署到Cloudflare Pages"
  },
  
  "code_standards": {
    "formatting": "使用eslint配置，遵循Next.js核心web vitals规则",
    "typescript": "严格类型检查，使用React组件的类型定义",
    "component_structure": "功能组件使用React Hooks，UI组件遵循shadcn模式"
  },
  
  "key_concepts": {
    "bookmarklet": "允许用户通过浏览器书签栏一键保存当前页面的JavaScript代码片段",
    "cloud_storage": "书签数据存储在Cloudflare KV中，允许跨设备访问",
    "internationalization": "支持多语言（英文和中文），使用URL查询参数切换",
    "edge_computing": "使用Cloudflare Edge运行时以获得全球低延迟访问"
  },
  
  "folder_structure": {
    "src/app": "Next.js App Router页面和API路由",
    "src/components": "React组件，包括UI组件和功能组件",
    "src/components/ui": "shadcn UI组件库",
    "src/lib": "工具函数和共享逻辑"
  },
  
  "ai_guidance": {
    "preferred_patterns": [
      "使用React Hooks管理状态",
      "使用Next.js App Router约定",
      "组件级的国际化",
      "使用Edge运行时优化性能",
      "遵循Cloudflare Workers模式操作KV存储"
    ],
    "avoid": [
      "使用客户端存储作为主要数据存储",
      "过度复杂的状态管理",
      "直接DOM操作",
      "大型内联样式"
    ]
  }
}

---
> Source: [wesleyel/cloudmark](https://github.com/wesleyel/cloudmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
