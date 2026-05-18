## videoadguard

> 本文件为 Claude Code (claude.ai/code) 提供代码库工作指南。

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供代码库工作指南。

## 构建和测试命令

```bash
npm install          # 安装依赖
npm run build        # 构建 Chrome 和 Firefox 扩展
npm run build:chrome # 仅构建 Chrome 扩展
npm run build:firefox # 仅构建 Firefox 扩展
npm run clean        # 清理构建产物

# 测试命令
npm test             # 运行所有测试（Vitest）
```

**Worker 部署命令：**

```bash
cd worker
npx wrangler deploy
```

部署地址：`https://videoadguard-api.0100320.xyz`

当你完成一项完整的修改后，运行构建命令，确认构建正确后便于我测试。

## Git 提交规则

**所有 commit 必须经过用户确认方可提交。** 完成修改后主动询问用户："是否需要提交 commit？"，用户同意后我才执行 `git commit`。如果用户有其他需求，一并提出后再提交。

## 项目架构

### 浏览器扩展结构
扩展采用多脚本架构：
- **content.ts**：主广告检测逻辑，运行在 B站视频页面
- **background.ts**：处理跨域请求（LLM API 调用、音频转录）
- **popup.ts/popup.html**：API 配置设置界面

### 广告检测流程
1. Content script 从 URL 提取视频 BV 号
2. 通过 BilibiliService 获取视频元数据、字幕和热门评论
3. **并行查询**：本地缓存 → 云端缓存（与 getVideoInfo 并行）+ 视频信息
4. 三条路径统一通过 `applyDetectionResult()` 初始化 UI（跳过按钮、标记层、自动跳过）
5. AI 检测完成后异步上传结果到云端（不阻塞主流程）
6. 用户手动拖拽调整广告区间后，可通过反馈按钮提交修正到云端

### 多 Provider LLM 支持
`src/services/llm/providers.ts` 中的 LLMGateway 将请求路由到不同 Provider：
- **openai**：OpenAI 兼容 API（302.AI、硅基流动等）
- **anthropic**：Claude API
- **custom_fetch**：本地 Ollama 及自定义端点

Provider 通过 API URL 自动识别。

### 浏览器特定音频实现
Webpack 通过别名替换 `./services/audio`：
- `audio.chrome.ts`：Chrome/Edge 音频处理
- `audio.firefox.ts`：Firefox 音频处理

### 云端缓存架构

**数据模型：**
- Key: `{bvid}`（每个视频一个主记录，直接使用 BV 号）
- `accuracy: 'accurate'` — 正常记录，查询时返回
- `accuracy: 'inaccurate'` — 用户标记不准确，查询时跳过
- `source: 'ai' | 'user'` — 来源（AI检测或用户手动修正）
- `model` / `provider` — 检测时使用的模型信息

**数据流：**
- 检测前并行查询云端缓存（Promise.allSettled）
- 查询时 Worker 仅返回 `accuracy === 'accurate'` 的记录
- 命中则跳过 AI 调用，加速用户体验
- AI 检测完成后异步上传（不阻塞正常流程）
- 用户反馈：
  - **标记不准确**：云端 accuracy 设为 'inaccurate'，本地标记 inaccurate
  - **手动修正**：云端用本地区间覆盖，仅更新 accuracy 和 source，保留其他字段
- 本地缓存准确时读取跳过（不等同于删除）

**API 端点（Worker）：**
- `GET /api/cache/:bvid` — 查询缓存（仅返回 accurate 记录）
- `POST /api/cache` — 保存/更新广告缓存

**安全措施：**
- BVID 格式校验：`^BV1[0-9A-Za-z]{8,}$`
- Rate Limiting（GET 2次/秒，POST 10次/分）
- 云端服务异常时静默降级，不影响本地检测

**Worker 部署：** `https://videoadguard-api.0100320.xyz`

### 关键技术细节
- **AV to BV 转换**：`BilibiliService.convertAvToBv()` 本地算法
- **WBI 签名**：Bilibili API 需要通过 `utils/wbi.ts` 签名参数
- **本地缓存**：24小时过期，`accuracy: 'inaccurate'` 时读取跳过，保留数据
- **云端缓存**：Cloudflare Worker + KV，永久保留
- **交互式广告标记**：点击切换激活状态，拖拽调整大小/位置
- **用户反馈**：标记不准确或提交修正，反馈按钮在检测完成后始终显示

## 项目结构

```
src/
├── background.ts          # Background script（跨域代理）
├── content.ts             # Content script（广告检测逻辑）
├── popup.ts/.html         # 设置界面
├── services/
│   ├── ai.ts             # AI 检测提示词和解析
│   ├── bilibili.ts       # Bilibili API 客户端
│   ├── cache.ts          # 本地检测结果缓存（24h TTL）
│   ├── cloud-cache.ts    # 云端缓存服务（Cloudflare KV）
│   ├── whitelist.ts      # UP主白名单
│   ├── audio.ts          # 音频转录（浏览器特定实现）
│   └── llm/              # LLM Provider 抽象层
│       ├── config.ts     # Provider 自动识别
│       ├── providers.ts  # OpenAI/Anthropic/custom_fetch
│       └── types.ts      # LLM 类型定义
├── types/
│   └── cloud-cache.ts    # 云端缓存和反馈类型定义
└── utils/
    ├── wbi.ts            # Bilibili WBI 签名
    ├── errors.ts         # 错误规范化
    └── logger.ts         # 日志工具

worker/                   # Cloudflare Worker（云端缓存网关）
├── src/index.ts         # Worker 入口，API 路由
├── wrangler.toml       # Worker 配置（KV namespace binding）
├── package.json         # Worker 依赖（wrangler）
└── README.md            # 部署文档
```

---
> Source: [Warma10032/VideoAdGuard](https://github.com/Warma10032/VideoAdGuard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
