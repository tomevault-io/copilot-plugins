## framebaker

> 像素风逐帧动画编辑器（Bun 全栈）。素材导入（GIF/MP4 拆帧、PNG、外部 CLI 生成）→ 帧编辑（PixiJS 洋葱皮）→ 时间轴排序 → 播放预览 → 导出精灵帧。

# FrameBaker — AGENTS

像素风逐帧动画编辑器（Bun 全栈）。素材导入（GIF/MP4 拆帧、PNG、外部 CLI 生成）→ 帧编辑（PixiJS 洋葱皮）→ 时间轴排序 → 播放预览 → 导出精灵帧。

## Monorepo 结构（Bun workspaces）

```
apps/
  server/        @framebaker/server — Elysia API + 任务队列 + bun:sqlite；Bun.serve 托管前端
  web/           @framebaker/web   — React 19 + pixi.js v8 + motion + lucide-react；index.html 是打包入口
packages/
  shared/        @framebaker/shared — 前后端共享类型/常量（无构建，exports 直指 src/index.ts）
docs/            架构 / API / roadmap 文档
storage/         运行时生成（已 gitignore），固定解析到仓库根，与启动 cwd 无关
```

## 常用命令

```bash
bun install          # 安装全部 workspace 依赖（bun 用 isolated 布局，各包有自己的 node_modules）
bun dev              # 开发（--hot），http://localhost:3000，PORT 可覆盖
bun start            # 生产
bun run typecheck    # tsc -p apps/server && tsc -p apps/web，改动后必须通过
```

无测试框架；验证方式 = typecheck + curl 冒烟（见 docs/api.md 的示例）。

## 约定

- **不要执行任何 git 操作**（不 init / commit / push），除非用户明确要求。
- 共享类型、枚举、WS 事件名一律放 `packages/shared`（FRAME_STATUSES / FRAME_SOURCES / JOB_TYPES / WS_EVENTS / SOURCE_COLORS / Frame / FramePatch 等），前后端都从这里导入，不要在 web 里重新定义。
- 后端文件路径必须用 `db.ts` 导出的 `STORAGE_ROOT`（基于 import.meta.dir），禁止依赖 cwd 的相对路径。
- 依赖最小化：不引入新依赖除非确有必要；拖拽用原生 HTML5 DnD，不装 dnd 库；不用 Vite / react-router / drizzle。
- 任务依赖保持 `queue.ts` → `jobs/*` 单向；worker 需要创建后续任务时由调度层注入窄回调，禁止反向导入 `queue.ts`。
- 外部命令（ffmpeg / 生成 CLI / 抠图 CLI）一律走 `apps/server/src/jobs/run.ts` 的 runCmd（Bun.spawn + stderr 捕获），命令模板按空白 split 后替换占位符，禁止拼 shell 字符串。
- 生成/抠图/提示词加强的运行配置经 `apps/server/src/provider.ts` 解析：**settings 表（设置页）优先，环境变量兜底**，每次调用实时读取，不要缓存启动时值。生成 provider 是列表，连接凭证与能力分层：`imageModels` / `videoModels` / `textModels`，尺寸为 `imageSize` / `videoSize`（旧 `apiModels` / `apiSize` 仅输入兼容）；提示词增强器只保存 `providerId + model` 并复用 api/dashscope 凭证（旧独立凭证兼容）。生成执行 adapter 在 `apps/server/src/providerAdapter.ts`；API 系协议走 `apps/server/src/jobs/generateApi.ts`；产物入库由 `generatedArtifacts.ts` 统一处理。**CLI 一律用结构化字段**（`cliBin` + 参数字段，服务端组 argv），不要在设置页引入手写模板。体检与联通测试在 `doctor.ts`。
- 前端图像重活（剪裁解码/透明边扫描/PNG 编码）走 `apps/web/src/imageops/` 的 Web Worker（OffscreenCanvas；脚本经服务端路由 `/imageops/imageOps.worker.js` 按需 Bun.build 下发，不要用 `new Worker(new URL(...))` —— Bun HTML 打包不处理），worker 不可用时自动降级主线程 canvas（纯计算在 `ops.ts`，两侧共用）；剪裁 UI 在 `components/CropModal.tsx`，导入弹窗的逐张剪裁队列在 `hooks/useCropQueue.ts`，文件状态/上传/任务收尾在 `hooks/useImportWorkflow.ts`。
- 帧变换语义统一为「图片中心锚点 → offset 像素平移 → rotation 弧度旋转 → scale 等比缩放 → opacity 合成」；纯几何唯一事实源在 `apps/web/src/frameGeometry.ts`，Pixi 编辑/预览与 `export.ts` 必须使用它保持一致，精灵帧导出把变换烘焙进共享原点的统一单元格。
- UI 文案与代码注释用中文；用户可见文案一律走 `apps/web/src/i18n.ts` 的 `t()` / `useT()`（中文即字典 key：`t("新建项目")`，zh 直接返回 key，en 查 `apps/web/src/i18n/en.ts`，缺失回退 key；插值用 `{name}`）。界面语言 zh/en 持久化同主题：localStorage `framebaker-lang` 首屏防闪烁 + settings 表 `lang` 权威（`LangToggle` 挂在 `TopNav`，`index.html` 内联脚本预读）。日期用 `getLocale()`（`zh-CN` / `en-US`）。素材/项目支持多级文件夹（`folders` 表 + `folder_id`，UI 左树 `FolderTree`）；生成来源按 provider 类型写入 `source`（`cli`/`api`/`dashscope`/`gemini`/`minimax`…）；任务可 `POST /api/jobs/:id/cancel` 取消。像素风主题（Fusion Pixel 12 字体、box-shadow 阶梯边框、image-rendering: pixelated），配色为 Cassette Futurism 双主题调色板（深色 Magnetic Night 默认 / 浅色 Beige Terminal），全部走 `apps/web/src/styles.css` 的 CSS 变量（`[data-theme="dark"|"light"]`），不要新增硬编码色值；主题管理在 `apps/web/src/theme.ts`。
- 不用浏览器默认弹窗（alert/confirm/prompt）：错误/提示走 `apps/web/src/notice.ts` 的 `notify()`，确认走 `await askConfirm()`，渲染由 App 根部 `AppModals` 单例完成。
- 改动 API 时同步更新 `docs/api.md`；改动架构/目录结构时同步更新 `docs/architecture.md` 与本文件。
- `storage/` 与 `node_modules/` 已 gitignore；smoke test 后清理 storage 与 /tmp 临时文件。

## 环境变量

- `PORT`（默认 3000）
- `FRAMEBAKER_GEN_CLI`：CLI 生成模板，占位符 `{prompt}` `{output}` `{index}` `{reference}` `{model}`（引用图由前端传 referenceMaterialId/referenceFrameId，服务端按 id 解析路径，模板与引用图不一致在创建 job 时 400）。**兜底项**：设置页可配多个生成 provider（CLI 模板 / OpenAI 兼容 API，存 settings 表 `genProviders`，生成时按 id 选择、模型单独指定）；仅当 provider 列表为空时本 env 合成为 id=`env` 的 CLI provider
- `FRAMEBAKER_MATTING_CLI`：自定义抠图模板，占位符 `{input}` `{output}`（可选 `{model}`）；**兜底项**，设置页结构化字段（matting.cliBin + 参数名）优先
- `FRAMEBAKER_MATTING_MODEL`：rembg 模型名（默认 `u2net`）；**兜底项**，设置页 matting.model 优先；模型缓存在 `storage/models`（U2NET_HOME）
- 抠图引擎：未配 CLI 时用 `scripts/setup_matting.sh`（Windows 用 `scripts/setup_matting.ps1`）安装的 `.venv-matting` 内置 rembg（POSIX 为 `bin/rembg`，Windows 为 `Scripts/rembg.exe`；已 gitignore），再次之 PATH rembg，最后 passthrough 复制；探测结果见 `GET /api/config`（每次请求实时解析）

---
> Source: [taotao7/FrameBaker](https://github.com/taotao7/FrameBaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
