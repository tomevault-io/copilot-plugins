## deepread

> 沉浸式 EPUB 阅读器，核心理念：深度思考而非消费内容。

# Deepread 深读

沉浸式 EPUB 阅读器，核心理念：深度思考而非消费内容。

## 产品逻辑链

上传 EPUB → 阅读意向仪式（"为什么打开这本书？"）→ 上下滚动阅读 → 拖拽摘录生成思考卡片 → AI 三种模式反思（释义/反驳/联想）→ 读完的书变成「知识沉淀」堆叠

## 技术栈

- React 19 + TypeScript + Vite
- Express 5（server.ts，API 代理）
- epub.js（CDN 加载，全局 `ePub`）
- framer-motion（动画）
- IndexedDB（本地持久化：books / content / cards）
- Tailwind CSS（CDN）
- OpenAI 兼容 Chat Completions API（通过通用 `AI_*` 环境变量配置）

## 项目结构

```
deepread/
  index.html          # SPA 入口，CDN 加载 Tailwind/epub.js/fonts
  index.css           # epub iframe 样式
  index.tsx           # React root mount
  App.tsx             # 根组件，状态机：ENTRY → DESK → RITUAL → READING
  server.ts           # Express 5，POST /api/ai/reflect 代理 OpenAI 兼容接口
  types.ts            # AppState / Book / ThoughtCard 类型定义
  constants.ts        # 占位（mock 数据已清理）
  utils/db.ts         # IndexedDB 封装（DeepreadDB v2）
  components/
    EntryView.tsx      # 入口仪式页（夜景点灯动效，无认证）
    DeskView.tsx       # 书桌视图，EPUB 上传，活跃书+沉淀堆叠
    RitualView.tsx     # 阅读意向仪式
    ReadingView.tsx    # EPUB 阅读器 + 思维流卡片面板
```

## 启动

```bash
npm run dev    # tsx server.ts，端口 3000，Vite 中间件模式
npm run build  # vite build + esbuild 打包 server
npm start      # 生产模式 dist/server.cjs
```

## 环境变量

`.env.local` 中配置：
- `AI_API_KEY` — 使用 AI 功能时必填
- `AI_API_BASE` — 默认 OpenAI 兼容端点
- `AI_MODEL` — 默认 gpt-4o-mini

## 当前进度

### 阶段一（核心闭环）— 基本完成

- [x] EPUB 渲染：paginated 模式，滚轮翻页，800ms 防抖
- [x] 位置恢复：CFI 持久化到 IndexedDB，下次打开恢复到上次位置
- [x] 真实进度：`book.locations.generate(1024)` 后用 `percentageFromCfi` 计算百分比
- [x] 完成判定：progress >= 95 时标记为 100，书进入沉淀堆叠
- [x] 进度条 UI：书名旁显示细进度条、百分比、当前章节名
- [x] 目录侧栏：点击目录按钮弹出章节目录，可跳转
- [x] IndexedDB：连接池复用，updateBookMetadata O(1) 直接 key 查找
- [x] 补 index.css / 清理死代码 / 修复 server.ts 智能引号
- [x] 代码质量审计修复（闭包泄漏、stale state、多余写入等）

### 待做

- [x] 书籍删除功能（删除书籍时同步清理内容和卡片）
- [x] 书籍封面提取（失败时回落到封面颜色）
- [x] 思维流卡片管理（删除卡片）
- [x] 阅读完成回顾（读完时回看当初意图 vs 收获）
- [x] 导出能力（卡片导出为 Markdown）
- [ ] AI 反思质量优化（prompt 长度策略、流式输出）
- [ ] 阅读统计（累计时长、卡片数量）
- [ ] 全文/卡片搜索

## 约束

- 纯桌面端应用，不考虑移动端
- 不部署 AI Studio，本地运行
- 入口仪式为纯 UI 动效，无认证/无账号密码
- epub.js 通过 CDN `<script>` 加载，用 `(window as any).ePub` 访问

---
> Source: [AyingAI/deepread](https://github.com/AyingAI/deepread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
