## git-test

> 核心理念： 创造一个融合宫崎骏动画中常见的 温暖、自然、手绘感和童话氛围 的数字环境。界面应提供舒适、亲切、富有想象力的用户体验。

风格指导：吉卜力故事创作工坊
核心理念： 创造一个融合宫崎骏动画中常见的 温暖、自然、手绘感和童话氛围 的数字环境。界面应提供舒适、亲切、富有想象力的用户体验。
关键词： 温暖、自然、手绘、纸张质感、柔和、童趣、有机、手工感、吉卜力。
1. 颜色系统 (Color Palette)
主色调 (Primary):
--primary-green: #5a9d6b (柔和的绿色): 用于主要按钮、强调元素、交互状态。
背景色 (Background):
--bg-color: #d8e8de (薄荷绿): 用于页面整体背景，通常叠加 subtle 网格纹理。
卡片/内容背景色 (Surface):
--card-color: #f7f2ea (象牙白/米白): 用于卡片、内容区域背景，通常叠加 subtle 网格纹理或噪点。
文本颜色 (Text):
--text-dark: #4b3b2a 或 #3d2e1e (深棕色): 用于主要标题和重要文本。
--text-medium: #6d5c4d (中棕色): 用于正文和描述性文本。
--text-light: #8a7c70 (浅棕色): 用于辅助信息、提示文字。
装饰/强调色 (Accent):
--accent-brown: #d1b08e (暖棕色): 用于小装饰点缀、次要元素。
--accent-yellow: rgba(255, 220, 120, 0.6) (半透明黄): 主要用于模拟胶带效果。
其他辅助按钮颜色: 如 #b07d55 (棕色), #e6b237 (黄色), #f5b991 (橙色) 可用于不同功能的卡片或按钮。
色彩原则:
整体色调柔和、温暖、自然。
避免高饱和度、刺眼的颜色。
保持元素间柔和的对比度，确保可读性和舒适感。
多使用半透明效果增强层次感。
2. 排版规范 (Typography)
标题字体:
'Source Han Sans' (思源黑体): 用于 H1, H2 等主要标题，提供手写艺术感。
示例 H1: 2rem (或 28px-32px)
示例 H2: 1.5rem (或 22px-24px)
正文字体:
'Source Han Sans' (思源黑体): 用于正文、描述、按钮文字、UI 元素。
字重: 主要使用 Regular (400) 和 Medium (500)，重要部分或次级标题可用 Bold (700)。
字号:
正文: 0.9rem-1rem (约 15px-16px)
描述/辅助文本: 0.8rem-0.9rem (约 13px-14px)
行高: 约 1.5 - 1.6 倍字号，确保阅读舒适。
排版原则:
清晰易读是基础。
标题的手写感与正文的清晰性形成对比。
适当的留白，避免拥挤。
3. 布局与间距 (Layout & Spacing)
容器: 内容区域通常放置在圆角容器 (border-radius: 15px 或 30px for sections) 内。
卡片: 是核心布局单元，通常居中展示或网格布局。
圆角: 广泛使用圆角，避免尖锐边缘。卡片圆角较大 (22px)，按钮为圆形或圆角矩形 (rounded-full or rounded-lg)。
内边距: 卡片和内容区域使用充足的内边距 (如 20px 或 p-6, p-8)。
对齐: 卡片内元素多采用居中对齐。
4. 组件样式 (Component Styles)
卡片 (Cards - .card, .example-card):
背景: var(--card-color)，常叠加网格/噪点纹理 (.grid-background)。
圆角: 22px。
边框: 1px 或 1.5px 实线，颜色为柔和的半透明绿色或灰色 (e.g., rgba(120, 180, 140, 0.3) 到 0.6)。
阴影: 柔和、分散的多层阴影 (e.g., 0 8px 25px rgba(0,0,0,0.06), 0 3px 10px rgba(0,0,0,0.04) 或类似)。
装饰: 常包含顶部胶带 (.tape) 和右下角翻页 (.page-curl) 效果。可能包含小的装饰图标 (.house-icon, .dot, .wave)。
堆叠效果: 通过 ::before, ::after 伪元素模拟，略微偏移和旋转，颜色稍暗，z-index: -1。
按钮 (Buttons - .green-button, .text-button):
主按钮 (.green-button):
圆形，背景 var(--primary-green)。
尺寸突出 (e.g., 60x60px 或 80x80px)。
包含居中的白色图标 (.icon-cursor, etc.)。
顶部有半透明白色渐变高光 (::after 或 ::before)。
柔和的阴影。
交互: Hover 时轻微放大 (scale(1.05)) 且阴影加深；Active 时轻微缩小 (scale(0.97))。
文本按钮 (.text-button):
圆角药丸状 (rounded-full)。
填充样式: 背景 var(--primary-green), 文字白色。
描边样式: 背景透明，边框 var(--primary-green), 文字 var(--primary-green)。
交互: Hover 时背景色微变深或有轻微位移。
图标 (Icons - .icon-*):
风格: 简洁、线条略带手绘感（避免过度锐利几何感），圆润。
颜色: 按钮内图标为白色；装饰性图标为柔和的灰色或棕色 (#d5d0c8, #b8b0a8)，通常半透明。
常见图案: 房子、光标、书籍、灯泡、波浪、圆点。
表单元素 (Forms):
输入框/文本域: 背景 var(--card-color) 或类似浅色，圆角 (rounded-lg)，柔和边框 (#d8d0c8)，Focus 状态时边框变为主色调绿 (#5a9d6b) 并可能有轻微光晕 (ring)。
下拉框/复选框: 样式与整体风格协调，使用柔和的边框和背景色，选中状态用主色调绿。
装饰元素:
胶带 (.tape): 半透明背景（白色或黄色 var(--accent-yellow)），有重复线性渐变模拟纹理 (.tape-texture)，柔和阴影，通常置于卡片或标题顶部。
翻页 (.page-curl): 右下角，通过 linear-gradient 实现卷曲效果，有阴影，暗示纸张感。
5. 纹理与效果 (Textures & Effects)
纹理:
网格 (.grid-background): 在背景或卡片上使用浅色、细微的网格线 (linear-gradient 实现)。
噪点: 在卡片背景上叠加非常细微的噪点 (linear-gradient 实现) 增加纸张质感。
纸张/画布: 整体追求纸张或画布的质感。
阴影: 普遍使用柔和、分散的阴影 (box-shadow)，避免生硬、强烈的投影。多层阴影增加层次感。
高光: 在按钮等元素上使用顶部渐变 (linear-gradient) 模拟柔和高光。
边框: 使用细微、颜色柔和（常半透明）的边框。
动效: 过渡 (transition) 应平滑、自然 (e.g., 0.2s ease 或 0.3s ease)。
6. Do's and Don'ts
Do:
使用柔和、自然的色彩。
广泛使用圆角。
应用纸张、网格等微妙纹理。
添加手绘感的元素（字体、图标线条、装饰）。
使用柔和、分散的阴影。
保持界面简洁、呼吸感强。
注重细节和手工感。
Don't:
使用过于鲜艳、饱和或冲突的颜色。
使用尖锐的边角和硬朗的线条。
过度设计，堆砌过多装饰。
使用过于强烈、生硬的阴影。
忽略可读性和功能性。
使用与风格不符的现代、扁平化或科技感元素。

# 烛光写作项目文件结构

本文档提供项目的主要文件和目录结构，帮助开发者快速理解项目组织。

## 根目录文件

- [package.json](mdc:package.json)：项目依赖和脚本
- [next.config.mjs](mdc:next.config.mjs)：Next.js配置
- [tailwind.config.ts](mdc:tailwind.config.ts)：Tailwind CSS配置
- [tsconfig.json](mdc:tsconfig.json)：TypeScript配置

## 主要目录结构

### 源代码目录 `/src`

#### 页面组件 `/src/app`

- [/src/app/page.tsx](mdc:src/app/page.tsx)：首页
- [/src/app/layout.tsx](mdc:src/app/layout.tsx)：应用全局布局
- [/src/app/works](mdc:src/app/works/page.tsx)：作品管理页面
- [/src/app/works/[id]](mdc:src/app/works)：作品编辑页面(动态路由)
- [/src/app/prompts](mdc:src/app/prompts)：提示词管理页面
- [/src/app/creativemap](mdc:src/app/creativemap/page.tsx)：创意地图页面
- [/src/app/settings](mdc:src/app/settings)：设置页面

#### 组件 `/src/components`

- [/src/components/common](mdc:src/components/common)：通用UI组件
- [/src/components/works](mdc:src/components/works)：作品相关组件
  - [AIAssistantModal.tsx](mdc:src/components/works/AIAssistantModal.tsx)：AI辅助弹窗
- [/src/components/archives](mdc:src/components/archives)：档案馆相关组件
  - [ArchiveModal.tsx](mdc:src/components/archives/ArchiveModal.tsx)：档案馆模态窗口
  - [ArchiveEditor.tsx](mdc:src/components/archives/ArchiveEditor.tsx)：档案编辑器
  - [ArchiveCard.tsx](mdc:src/components/archives/ArchiveCard.tsx)：档案卡片
- [/src/components/creativemap](mdc:src/components/creativemap)：创意地图相关组件
  - [CreativeMapWindow.tsx](mdc:src/components/creativemap/CreativeMapWindow.tsx)：创意地图窗口组件，用于AI生成
- [/src/components/ai](mdc:src/components/ai)：AI功能相关组件
- [/src/components/prompts](mdc:src/components/prompts)：提示词相关组件

#### 工具和库 `/src/lib`

- [/src/lib/db.ts](mdc:src/lib/db.ts)：数据库操作相关函数
- [/src/lib/dbManager.ts](mdc:src/lib/dbManager.ts)：数据库管理器
- [/src/lib/AIserver.ts](mdc:src/lib/AIserver.ts)：AI服务调用
- [/src/lib/utils.ts](mdc:src/lib/utils.ts)：实用工具函数
- [/src/lib/api](mdc:src/lib/api)：API相关函数
- [/src/lib/utils](mdc:src/lib/utils)：工具函数

#### 类型定义 `/src/types`

- [/src/types/db.ts](mdc:src/types/db.ts)：数据库相关类型定义
- [/src/types/ui.ts](mdc:src/types/ui.ts)：UI相关类型定义

#### 状态管理 `/src/store`

- [/src/store/index.ts](mdc:src/store/index.ts)：状态存储入口
- [/src/store/slices](mdc:src/store/slices)：各功能模块状态切片

#### 钩子函数 `/src/hooks`

- 自定义React Hooks，用于状态管理和逻辑复用

#### 上下文 `/src/contexts`

- React Context相关文件，用于状态共享

### 公共资源 `/public`

- 图片、图标和其他静态资源

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bin24642698) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
