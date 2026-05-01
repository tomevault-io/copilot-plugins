## gpuroi

> - 核心 TypeScript 源码位于 `src/`。

# 仓库指南

## 项目结构与模块组织
- 核心 TypeScript 源码位于 `src/`。
- `src/core/`：切面计算逻辑、CPU/GPU 后端、数学工具与 WGSL 着色器（`slicer.wgsl`）。
- `src/renderer/`：Canvas 渲染实现与高层 `SliceRenderer` 编排。
- `src/types/`：共享类型定义与常量。
- `src/demo/`：Vite 演示入口（`index.html`、`bench.html`）与示例脚本。
- 测试与代码同目录，位于 `src/core/__tests__/`，文件模式为 `*.test.ts`。
- 产物目录：`dist/`（构建输出）与 `coverage/`（覆盖率报告）。

## 构建、测试与开发命令
- `npm install`：安装依赖。
- `npm run dev`：启动 Vite 开发服务器并运行 Demo 页面。
- `npm run build`：先执行 `tsc` 类型构建，再用 Vite 打包演示页面。
- `npm run preview`：本地预览 `dist/` 中的构建结果。
- `npm test`：单次运行 Vitest（CI 风格）。
- `npm run test:watch`：以 watch 模式运行 Vitest。
- `npm run test:coverage`：运行测试并生成 V8 文本/HTML 覆盖率报告。

## 代码风格与命名规范
- 语言：TypeScript（`strict: true`，ESM 模块）。
- 缩进：2 空格；保持当前无分号风格一致。
- 文件命名：kebab-case（如 `slice-renderer.ts`、`create-slicer.ts`）。
- 类型/类名：`PascalCase`；函数/变量：`camelCase`；常量：`UPPER_SNAKE_CASE`。
- `src` 内部导入优先使用别名 `@/`（已在 Vite/Vitest/TS 中配置）。

## 测试规范
- 测试框架：Vitest（开启 `globals`）。
- 覆盖率范围包含 `src/core/**`、`src/renderer/**`、`src/types/**`。
- 新增测试应放在对应核心逻辑附近，通常在 `src/core/__tests__/`。
- 测试文件必须以 `.test.ts` 结尾，命名建议表达行为（如 `projection.test.ts`）。

## 提交与 Pull Request 规范
- 当前工作区快照不包含 Git 历史；后续提交建议采用 Conventional Commits（如 `feat(core): add GPU plane culling`）。
- 每次提交应聚焦单一改动，并确保可构建、可测试（build + tests 通过）。
- PR 应包含：
  - 行为变化的清晰说明。
  - 对应 issue/任务链接（如适用）。
  - 测试证据（`npm test` 或 `npm run test:coverage`）。
  - 涉及 Demo 或渲染效果变更时提供截图/GIF。

## New 方案（Batch Bitmap 渲染）
- `New` 路径定义为 `BatchGPUSlicer.sliceToBitmap`：先 compute 切面，再直接在 WebGPU render pass 输出位图。
- 线渲染不再使用 `line-list`，改为“每条线段一个 instanced quad（三角形）”。
- 着色器位于 `src/core/slicer-batch-bitmap.wgsl`：
  - `build_draw_args` 输出 `vertexCount=6`、`instanceCount=segmentCount`。
  - 顶点阶段通过 `instance_index` 读取线段，并做法线方向扩张与端点延伸。
  - 关键参数：`LINE_HALF_WIDTH_PX`、`LINE_CAP_EXTEND_PX`（调平滑/粗细时优先改这里）。
- 渲染管线位于 `src/core/batch-gpu-slicer.ts`：
  - `primitive.topology = 'triangle-list'`。
  - 开启 `MSAA_SAMPLE_COUNT = 4`，并使用 MSAA 中间纹理 `resolve` 到输出纹理。
  - 当前为“无深度遮挡”模式：不启用 `depthStencil`，所有线段在同一平面叠加。
- `bench` 页面并发策略：
  - 内部使用串行队列（latest-wins）。
  - 入口统一走 `requestBenchmarkRun`，不要直接在事件回调并发调用 `runBenchmark`。
- 差异检测口径（`bench`）：
  - 当前比较的是像素 alpha mask（显示一致性），不是几何拓扑一致性。
  - 若要验证几何一致性，请优先比较 `Single` 与 `sliceBatchFlat` 的线段结果。

## Brush 方案现状（按 `src/core/brush/brush-example.ts` 思路落地）
- 2D 预览使用“圆刷/胶囊刷逐段 stamp + Clipper2 布尔”，不是旧的扫描线流程。
- `brushContourPoints` 当前固定默认 `40`（可配置），用于圆刷/胶囊轮廓采样。
- `erase` 必须按“面差集”语义处理闭环：
  - 闭环轮廓：`AddSubject + Difference`
  - 开口线段：`AddOpenSubject + Difference`
  - 输出合并闭环与开口结果，避免仅出现“边断开”。
- `DefaultBrushEngine2D` 为无 fallback 模式：必须提供 `clipperAdapter`；Clipper 执行失败直接抛错。
- `BrushSession.appendPoint` 使用增量预览（首帧后仅处理最后两点），并跳过 no-op 点，避免长笔画卡顿。
- `BrushSession` 在 `invalidated` 状态收到 `setBaseSegments` 后会自动回到 `idle`，切视角/改 anchor 后可继续勾画。
- Demo 当前也是无 fallback：仅支持 WebGPU Batch + Clipper2 预览路径。

---
> Source: [lsa2127291/gpuRoi](https://github.com/lsa2127291/gpuRoi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
