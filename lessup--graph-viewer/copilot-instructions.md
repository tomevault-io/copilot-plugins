## graph-viewer

> - Next.js App Router + React 19 + TypeScript 严格模式 + Tailwind CSS


# GraphViewer 项目规则

## 技术栈

- Next.js App Router + React 19 + TypeScript 严格模式 + Tailwind CSS

## 核心模块复用

优先复用现有核心模块，不要平行引入第二套状态流或渲染/导出逻辑：

- **状态层**: `useDiagramState`、`useSettings`、`useToast`
- **渲染层**: `useDiagramRender`、`useLivePreview`
- **动作层**: `useVersionActions`、`useWorkspaceActions`、`useAIActions`
- **配置层**: `lib/diagramConfig.ts`、`lib/types.ts`

## 类型约束

- `Engine` 和 `Format` 只能使用 `lib/diagramConfig.ts` 中定义的合法值，不要在业务代码里引入魔法字符串。
- 工作区图表对象结构 `DiagramDoc` 定义在 `lib/types.ts`，必须保持兼容：
  - `{ id, name, engine, format, code, updatedAt }`

## 引擎/格式/导出变更同步检查

修改图表引擎、格式或导出能力时，必须同步检查以下文件：

- `lib/diagramConfig.ts` — 类型定义、ENGINE_CATEGORIES、getKrokiType
- `lib/diagramSamples.ts` — 示例代码
- `lib/syntaxHighlight.ts` — CodeMirror 语言映射
- `lib/exportUtils.ts` — 导出格式映射
- `hooks/useDiagramRender.ts` — 本地/远程渲染逻辑
- `app/api/render/route.ts` — Kroki 代理、引擎白名单
- `components/EditorPanel.tsx` — 引擎/格式选择器 UI
- `components/PreviewPanel.tsx` — 预览渲染
- `components/PreviewToolbar.tsx` — 导出入口
- `app/page.tsx` — 顶层组合

## 编码风格

- UI 文案优先保持中文，与现有界面风格一致。
- 优先在现有组件上做小步修改，不要为了单点需求额外创建新的顶层页面结构或全局状态层。
- `catch` 块使用 `catch (e: unknown)` + `instanceof Error` 收窄，不要用 `catch (e: any)`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LessUp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
