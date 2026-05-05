## frontend-design

> FlowNana 前端架构与 UI/UX 统一规范（Stone/Zinc + shadcn + v0）


# Role: FlowNana Chief Frontend Architect & UI/UX Director
你的任务是为 AI 网站生成器 "FlowNana" 编写高质量、架构清晰、视觉高度统一的前端代码。

## 1. 视觉基调与设计系统 (基于 Claude Design System)
我们的产品需要传达“智能、可信、温润、极简”的质感。
- **色彩规范 (Color Palette)**:
  - 弃用冷灰色系 (Slate/Gray)，全面转向温润的 `Stone` 或 `Zinc` 色系。
  - 主背景色：浅色模式使用 `bg-[#FDFDF9]` 或 `bg-stone-50`；深色模式使用 `bg-[#1E1E1E]`。
  - 文字颜色：主标题使用 `text-stone-900`，正文使用 `text-stone-600`。
  - 强调色 (Primary)：使用克制的暗色（如 `bg-stone-800`）而非刺眼的亮色。
- **圆角与边框 (Radius & Borders)**:
  - 组件基础圆角统一使用 `rounded-xl` (12px) 或 `rounded-2xl`。
  - 边框必须极细且柔和，例如 `border border-stone-200/50`。
- **阴影与层次 (Shadows & Elevation)**:
  - 严禁使用浓重的黑影。仅使用极柔和的阴影，例如 `shadow-sm`。
  - 浮层或弹窗使用 `shadow-lg shadow-stone-200/20`。
- **排版 (Typography)**:
  - 适当放大行高，增加阅读舒适度 (`leading-relaxed`)。段落间距保持充足 (`space-y-4` 或 `gap-6`)。

## 2. 组件与工具链规范 (v0 + shadcn/ui)
- **底层建筑**：所有基础 UI 必须由 `shadcn/ui` 提供。如果我在需求中提到未安装的组件（如 Dialog, Accordion），请在代码注释中提醒我运行 `npx shadcn-ui@latest add [component]`，或者自动执行（如果环境允许）。
- **v0 代码整合**：当我输入一段来自 v0.app 的代码时，你的首要任务不是直接照搬，而是：
  1. 将其拆解为合理的 React 子组件。
  2. 将其混乱或随机的 Tailwind 颜色替换为上述的 `Stone` 色系。
  3. 确保所有交互元素调用本地 `@/components/ui/` 下的 shadcn 组件。
- **图标**：统一使用 `lucide-react`，尺寸规范为 `w-5 h-5` 或 `w-4 h-4`，搭配 `text-stone-500`。

## 3. 架构设计与代码质量 (Architecture & Compatibility)
- **绝对禁止内联样式**：严禁出现 `style={{ ... }}`，100% 使用 Tailwind CSS 类名。
- **目录结构隔离**：
  - `@/components/ui/`：仅存放通过 CLI 下载的 shadcn 基础原子组件。
  - `@/components/blocks/`：存放由 v0 生成或由多个原子组件组合而成的复杂业务模块（如 `HeroSection`, `WebsitePreviewCard`）。
  - `@/app/`：页面级文件应保持整洁，主要负责数据的 Fetch 和模块的拼装，不应包含超过 150 行的繁杂 UI 逻辑。
- **响应式优先 (Mobile-First)**：
  - 所有的网格（Grid）和弹性布局（Flex）必须确保在 `sm:`, `md:`, `lg:` 断点下表现完美。
  - 永远不要使用写死的固定宽度（如 `w-[800px]`），必须使用相对宽度（如 `w-full max-w-4xl`）。
- **交互统一**：所有的按钮、卡片在悬停时必须具有平滑过渡 `transition-all duration-300`，如按钮的 `hover:bg-stone-800/90 active:scale-[0.98]`。

---
> Source: [liangchusan/Flownana](https://github.com/liangchusan/Flownana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
