## fragment-universe

> 本文件是 Codex 在开发《碎片宇宙》时必须遵守的项目规范。修改代码前先阅读 `README.md`、`PRD.md`、`ARCHITECTURE.md` 和本文件。代码、需求与文档冲突时，先指出冲突或同步文档，不得自行猜测。

# AGENTS.md

本文件是 Codex 在开发《碎片宇宙》时必须遵守的项目规范。修改代码前先阅读 `README.md`、`PRD.md`、`ARCHITECTURE.md` 和本文件。代码、需求与文档冲突时，先指出冲突或同步文档，不得自行猜测。

## 1. 产品边界

当前 V1 只包含：暗淡且不可点击的环境星空；文字、链接和图片记录；正式 Fragment 成功创建后点亮稳定内容星；点击星星打开真实 Fragment；追加 FragmentNote 后让目标星在有限范围内变亮；内容星数量增加后让星空以有上限的速度略微加快；IndexedDB 本地存储、完整备份与恢复；GetNote 链接读取和 OpenAI 图片分析的成功、失败边界。

不进入 V1：关系线、知识图谱、星座、星团、语义聚类、AI 选星或布局、推荐、通用聊天、账号、云同步、社交、积分和经营玩法。

## 2. 技术约束

必须使用 TypeScript strict、Next.js App Router、React、Tailwind、Three.js、Dexie/IndexedDB、Route Handlers、Zod、Vitest、Playwright 和 npm。

未经确认不得引入 Redux、独立 Express 后端、云数据库、Electron、Tauri、React Three Fiber、物理引擎、大型 UI 组件库或其他 3D/游戏框架。

Three.js 必须通过项目依赖加载，不使用 CDN。主入口最多运行一个 WebGL Canvas。

## 3. 架构边界

### React

负责页面、表单、弹窗、详情、星空挂载、错误降级和导入导出。不得每帧同步星星位置，不得复制 Domain 的稳定坐标、亮度或转速规则，不得把正文交给渲染层。

### Three.js

负责环境星和内容星 GPU 渲染、命中检测、受限的 `fragmentId` 回调、自动旋转、选中反馈、resize、DPR、visibility、reduced motion 与完整资源清理。不得直接读写 Dexie、调用 Provider、读取正文或 Blob、决定关系、分类或重要度。

### Domain

Fragment 投影、稳定坐标、亮度、转速、删除过滤和 reduced-motion 静态规则必须放在 `domain/`，使用纯 TypeScript 和纯函数，不依赖 React、Three.js 或 Dexie。

### Data

数据库访问必须通过 Repository、Query、Service 和明确事务。兼容表与稳定存储标识不得在没有独立迁移、备份与恢复验证的情况下删除或改名。

### Provider

OpenAI 和 GetNote 只能通过 Next.js Route Handler。Key 只能放在服务端环境变量，不得进入客户端 bundle、IndexedDB、日志、测试快照或 Git。

## 4. 产品硬规则

1. 环境星不代表记录且不可点击。
2. 每个非 deleted 的正式 Fragment 最多对应一颗内容星。
3. processing job 不是记录，不得提前点亮星星。
4. 内容星刷新和备份恢复后位置必须稳定。
5. 追加只新增 FragmentNote，不覆盖原始正文。
6. 新建成功的独立追加都增加亮度，亮度必须有上限。
7. 内容星数量只以有上限函数影响运动速度。
8. reduced motion 停止自动旋转、脉冲和形变，保留静态差异。
9. 点击内容星必须打开真实 Fragment，不得展示 Mock 内容。
10. Provider 失败不得创建虚假 Fragment 或内容星。
11. WebGL 失败时内容和备份仍可访问。
12. 不新增 Star 表；星空由 Fragment 与 FragmentNote 派生。
13. 不得为视觉重构删除或破坏历史用户数据。
14. AI 不参与星空结构、颜色、大小、位置、关系或价值判断。

## 5. 创建链路

文字：本地校验 → 原子创建正式 Fragment → 真实内容星出现。

链接/图片：本地校验和独立披露 → 创建 ContentProcessingJob → Provider 成功 → 原子创建正式 Fragment；图片同时创建 MediaAsset。失败保留可重试任务，不创建虚假记录。

图片只接受 PNG/JPEG/WebP、单张且不超过 10 MB。Blob 保存在 job，成功后移动到唯一主 MediaAsset，取消时删除临时 Blob，不保存 Base64。

## 6. 数据规则

- `Fragment` 是内容星唯一事实来源，`starId === fragmentId`。
- `FragmentNote` 是追加数量和亮度事实来源。
- 不持久化星星坐标、亮度、颜色、尺寸或转速。
- 坐标由 `fragmentId + layoutVersion` 的纯函数派生。
- deleted Fragment 不进入投影。
- 时间使用 ISO 字符串，ID 使用 UUID。
- Schema 变化必须显式增加 Dexie 版本。
- 备份必须覆盖当前完整 Schema，恢复不得丢失未知兼容表。

## 7. 代码规范

- 禁止无理由 `any` 和 `as any`。
- API 输入输出使用 Zod。
- Domain 函数显式声明参数和返回类型。
- React 组件 PascalCase；函数变量 camelCase；常量 UPPER_SNAKE_CASE。
- 单个业务文件尽量不超过 300 行；大组件按明确职责拆分。
- 注释解释原因、边界和兼容性，不逐行翻译代码。
- 手工编辑使用 `apply_patch`。
- 不覆盖或回退未知用户改动。

## 8. UI 与可访问性

- Canvas 是主视觉，但创建、详情、设置与备份不能依赖 Canvas 成功。
- 异步操作必须展示 pending、success 或 failed。
- 错误文案必须说明内容是否保存。
- 删除、清空与覆盖恢复必须二次确认。
- 选中、展开、追加和保存反馈必须短、轻、清楚。
- 所有对话框支持关闭、Escape、焦点圈定和焦点恢复。
- 文本不得被动效遮挡或溢出容器。

## 9. 安全

- 链接仅允许 HTTP/HTTPS，拒绝 localhost、私有 IP 与内网地址。
- 设置超时和响应体上限，不执行脚本，不下载页面外资源。
- 上传限制类型、大小和尺寸。
- 不记录 API Key、完整用户正文、图片 Base64 或隐私内容。
- `.env.local` 必须被忽略且不得跟踪。

## 10. 测试

每项功能完成前至少运行：

```bash
npm run format:check
npm run lint
npm run typecheck
npm run test
```

涉及主流程时还要运行相关 Playwright 测试和 `npm run build`。优先覆盖稳定星位、亮度上限、转速上限、正确 Fragment 命中、单 Canvas、资源销毁、resize、visibility、reduced motion、保存失败边界、链接/图片成功前无星、备份恢复与兼容数据保留。

真实 Provider 不进入自动化测试；Mock 通过不等于真实 Provider 验收通过。

## 11. Git 与交付

- 任务开始先检查 `git status` 和当前分支。
- 不提交 `.env.local`、本地数据、测试报告或隐私截图。
- 未经用户明确授权，不得 push、改写历史、创建 Tag、Release 或部署。
- 用户授权历史重写时，仍必须先创建项目外恢复 bundle，并使用带租约的强制推送。
- 完成后汇报修改文件、验证结果、风险、提交、推送与远端状态。

---
> Source: [Zev55555/fragment-universe](https://github.com/Zev55555/fragment-universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
