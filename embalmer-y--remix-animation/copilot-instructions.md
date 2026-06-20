## remix-animation

> name: remix-animation

---
name: remix-animation
description: 'Generate motion graphics and animation videos with Remotion. Use when the user asks for 动画, 动效, video, motion graphics, transition, 片头, GIF animation, GSAP animation, or wants to turn static images, SVG, or text into motion. Search open-source references, adapt them to Remotion, render the result, and save reusable local templates.'
argument-hint: 'Describe the animation, duration, resolution, fps, output format, and any GSAP or reference link if available.'
---

# Skill: remix-animation

> 基于 Remotion 官方 prompt-to-motion-graphics 模板改造，加入"开源社区检索+Agent改编"能力，
> 让 Agent 能从 GSAP/Three.js/Lottie 等开源社区找到参考动画并改编为 Remotion 视频输出。

## 默认目标

- resolution: 默认 1440x1440（至少 1K；用户未指定时优先高分辨率）
- fps: 默认 120（最低 60；高复杂度或受限环境可降到 60）

## 资源引入

进入实现前，优先加载以下资源：

- [Remotion system prompt](./prompts/remotion-system-prompt.md)
- [GSAP skills / GSAP to Remotion rules](./rules/gsap-to-remotion.md)

如果用户请求明确涉及 GSAP、CodePen、timeline、ScrollTrigger 或已有 GSAP demo，必须同时读取上面的 GSAP 规则文档，再决定走“代码转换”还是“直接录制”路径。

## 触发条件

当满足以下任一条件时激活此 skill：

- 用户要求制作动画、视频、Motion Graphics
- 用户描述了一种具体的动画效果（如"粒子爆炸"、"文字飞入"、"3D地球旋转"、"数据图表动画"）
- 用户要求将静态内容（图片、SVG、文字）变为动态
- 触发关键词：动画、animation、motion、视频特效、动效、GIF、片头、转场

## 架构基础

本 skill 基于以下现有项目构建，**不重复造轮子**：

| 项目 | 作用 | 我们如何使用 |
|------|------|-------------|
| [create-vibe-motion](https://github.com/vibe-motion/create-vibe-motion) | Remotion 项目脚手架（npm: `create-vibe-motion`） | **作为兜底脚手架**：`scripts/init-workspace.sh` 优先复制仓库内置模板，缺失时才调用它 |
| [Remotion System Prompt](https://remotion.dev/docs/ai/system-prompt) | 教 LLM 写 Remotion 代码的标准 prompt | **作为 Agent 知识库**：[prompts/remotion-system-prompt.md](prompts/remotion-system-prompt.md) |
| [template-prompt-to-motion-graphics](https://github.com/remotion-dev/template-prompt-to-motion-graphics) | Remotion 官方 AI→动画 SaaS 模板 | **作为参考**（仅设计思路），不用来脚手架——我们优先使用仓库内置工作区模板 |
| [@remotion/three](https://remotion.dev/docs/three) | Three.js 官方适配 | **直接使用**：Three.js 动画无需手动转换 |
| [@remotion/lottie](https://remotion.dev/docs/lottie) | Lottie JSON 播放 | **直接使用**：Lottie 动画无需手动转换 |
| [remotion-animated](https://github.com/stefanwittwer/remotion-animated) | 声明式动画库 | **作为动画表达层**：简化动画编写 |
| [remotion-animation](https://github.com/ahgsql/remotion-animation) | animate.css → Remotion 桥梁 | **直接使用**：CSS 动画无需手动转换 |
| [gsap-video-export](https://github.com/workeffortwaste/gsap-video-export) | GSAP 动画逐帧录制导出视频 | **GSAP 路径 B**：无法代码转换时，直接录制 GSAP 动画为视频 |

> **为什么不用 GitHub clone `template-prompt-to-motion-graphics`？**
> 该仓库是 Vercel/Remotion 官方演示项目，依赖 SaaS 后端（OpenAI Key + Webhook），不适合本地命令行 Agent。
> `create-vibe-motion` 是社区维护的纯本地 Remotion 脚手架，npm 上一键 `npx` 创建，更适合 Agent 工作流。

### 我们的差异化能力（本 skill 新增）

官方模板只能让 AI "从零生成"动画代码。我们在此基础上增加：

```
📌 开源社区检索 → 找到相似动画参考
📌 Agent 智能改编 → 基于参考代码修改适配
📌 本地模板沉淀 → 改编成功自动保存，越用越快
📌 GSAP 双路径  → 代码转换 or 直接录制，灵活选择
```

---

## 前置条件

### 系统要求

- Node.js >= 18
- pnpm
- Chrome/Chromium，或本地 `chrome-headless-shell`（Remotion 渲染需要）
- ffmpeg（GSAP 录制 + 缩略图生成需要，可选）

### ⚠️ 不需要 OpenAI API Key

**与官方 `template-prompt-to-motion-graphics` 不同，本 skill 不依赖 OpenAI**。
所有代码生成与改编由 Agent 自己完成（Agent 本身就是 LLM），无需调用外部 LLM API。

### 环境初始化（仅首次，一行命令）

```bash
# Linux 全自动安装（Node.js / pnpm / Chromium 或 headless-shell / gsap-video-export / Remotion 工作区）
chmod +x scripts/setup-linux.sh scripts/init-workspace.sh scripts/smoke-test.sh
./scripts/setup-linux.sh

# 或手动（如果 Node.js 18+ / pnpm / 浏览器已就绪）：
./scripts/init-workspace.sh
```

补充说明：
- 仓库已自带 `remix-workspace/` 模板，初始化优先复用它。
- 无 sudo 的 Linux / WSL 环境会自动回退到本地 `chrome-headless-shell`。
- `ffmpeg` 不是基础 Remotion 渲染的硬依赖，但 `gsap-video-export` 和模板 GIF 缩略图需要它。

### 验证安装

```bash
chmod +x scripts/smoke-test.sh && ./scripts/smoke-test.sh
```

成功时会渲染一段测试动画到 `remix-workspace/out/smoke-test.mp4`。

---

## 执行流程

```
Step 1: 解析需求
Step 2: 检索本地模板
Step 3: 多源社区检索（如果本地未命中）
Step 4: 评估筛选
Step 5: 代码获取 + 改编
Step 6: 渲染输出
Step 7: 失败处理
Step 8: 沉淀为本地模板
```

---

### Step 1: 解析用户需求

从用户自然语言中提取结构化意图：

```json
{
  "animation_type": "",
  "style": "modern-minimal",
  "colors": [],
  "text": "",
  "duration_seconds": 5,
  "resolution": "1440x1440",
  "fps": 120,
  "output_format": "mp4",
  "reference_description": ""
}
```

**默认值**：未提及的字段使用默认值，不反复确认。若用户未给风格，默认使用“现代化简约（modern-minimal）”。若用户未给技术指标，默认目标为至少 1K 分辨率且 >60fps，优先 120fps。极度模糊时询问一次动画类型即可。

---

### Step 2: 检索本地模板库

检查 `local-templates/` 目录是否有匹配的已沉淀模板。

- 匹配方式：metadata.json 中的 tags 与用户需求语义匹配
- 匹配阈值：相似度 > 70% 视为命中
- **命中** → 跳到 Step 5 改编阶段
- **未命中** → 进入 Step 3

---

### Step 3: 多源社区检索

将用户描述转为英文搜索关键词，从以下来源并行检索（每源 Top 3）：

| 优先级 | 来源 | Agent 实际可用的检索工具 | 适合类型 |
|--------|------|------------------------|----------|
| 1 | GitHub | `github_text_search` / `github_repo` (跨 scope 搜)；`{kw} animation language:javascript sort:stars` | 通用 |
| 2 | CodePen | `fetch_webpage("https://codepen.io/search/pens?q={kw}&tag=gsap")` | DOM/SVG/CSS |
| 3 | NPM | `run_in_terminal` 跑 `npm search {kw} remotion` | Remotion 原生组件 |
| 4 | LottieFiles | `fetch_webpage("https://lottiefiles.com/search?q={kw}")` | 矢量动画/图标动效 |
| 5 | Shadertoy | `fetch_webpage("https://www.shadertoy.com/results?query={kw}")` | GPU 着色器特效 |

> **降级**：如果当前环境没有上述工具(例如纯 LLM 离线),Agent 应承认"无法检索",并直接进入 Step 5 分支 B(从零生成)。

#### Step 3 输出要求（硬门槛）

在进入 Step 5 之前，Agent **必须** 先给出一份最小检索记录；没有这份记录，不允许直接开始写动画代码。

最少应包含：

| 字段 | 要求 |
|------|------|
| search_keywords | 实际使用的英文关键词 |
| sources_checked | 实际访问的平台或 repo |
| candidates | 至少 2 个候选（若检索结果不足，则说明原因） |
| license | 每个候选的许可证状态：白名单 / 灰名单 / 黑名单 / 未知 |
| adaptation_notes | 为什么适合或不适合 Remotion 改编 |

**强制规则**：
- 没有许可证信息时，按“无许可证/黑名单”处理，不得直接改编
- 若使用 GitHub 结果，优先记录 repo URL、描述、星标、许可证、技术栈
- 若最终没有合规候选，必须先明确写出“无合规参考，因此转入从零生成”后，才能进入 Step 5 分支 B

#### 关键词生成示例

```
输入: "做一个数据从左飞入的柱状图动画"
输出: ["bar chart animation entrance", "d3 chart reveal gsap", "data visualization motion"]
```

---

### Step 4: 评估筛选

| 维度 | 权重 | 说明 |
|------|------|------|
| 视觉相似度 | 40% | 动画效果与用户需求匹配程度 |
| 代码可改编性 | 30% | 结构清晰、参数化程度高、单文件/少文件 |
| 许可证合规 | 20% | 白名单满分，灰名单半分 |
| 复杂度适中 | 10% | 过于复杂扣分 |

**许可证规则**：
- ✅ 白名单: MIT, Apache-2.0, ISC, BSD-2-Clause, BSD-3-Clause, CC0, Unlicense
- ⚠️ 灰名单: MPL-2.0, CC-BY-4.0
- ❌ 黑名单: GPL, LGPL, AGPL, CC-BY-NC, CC-BY-ND, 无许可证

**决策**：
- 最高分 >= 50% → 使用该参考，进入 Step 5
- 所有结果 < 50% → 进入 Step 5 的"从零生成"分支

#### Step 4 输出要求（硬门槛）

筛选结束后，Agent 必须明确产出：

```json
{
  "selected_reference": "repo/url/or null",
  "selection_reason": "",
  "license_status": "whitelist|graylist|blacklist|none",
  "score_breakdown": {
    "visual_similarity": 0,
    "adaptability": 0,
    "license": 0,
    "complexity": 0,
    "total": 0
  }
}
```

**禁止事项**：
- 禁止跳过 Step 3 / Step 4 直接生成代码
- 禁止仅凭“看起来像”或“我知道这个效果怎么做”就绕过检索
- 禁止使用黑名单或无许可证参考做改编起点

---

### Step 5: 代码获取 + 改编

#### 分支 A：基于参考代码改编

##### 5A.1 识别动画库并选择转换路径

| 参考代码使用的库 | 转换路径 | 说明 |
|----------------|----------|------|
| **Lottie** | 直接使用 `@remotion/lottie` | 无需转换，下载 JSON 即可 |
| **CSS / animate.css** | 使用 `remotion-animation` 包 | 已有现成桥梁 |
| **Three.js** | 使用 `@remotion/three` + R3F | 官方适配，改写 useFrame → useCurrentFrame |
| **GSAP（简单）** | 手动转换为 `interpolate()` | 按转换规则改写 |
| **GSAP（复杂）** | 使用 `gsap-video-export` 直接录制 | 不转换代码，Puppeteer 逐帧截图导出视频 |
| **Anime.js** | 转换为 `interpolate()` 或 `remotion-animated` | API 相近，转换简单 |
| **纯 Canvas/WebGL** | 改写为帧驱动 | 用 frame 替代 requestAnimationFrame |

##### 5A.2 GSAP 双路径策略

```
GSAP 动画代码
    ↓
评估复杂度
    ↓            ↓
  简单/中等       复杂（多层timeline/插件/ScrollTrigger）
    ↓            ↓
  代码转换路径    录制路径
  (interpolate)  (gsap-video-export)
```

**代码转换路径（GSAP 简单/中等）**：

```typescript
// 缓动映射
"none"        → Easing.linear
"power1.out"  → Easing.out(Easing.ease)
"power2.out"  → Easing.out(Easing.quad)
"power3.out"  → Easing.out(Easing.cubic)
"power4.out"  → Easing.out(Easing.quart)
"back.out"    → Easing.out(Easing.back)
"elastic.out" → Easing.out(Easing.elastic)
"bounce.out"  → Easing.out(Easing.bounce)

// gsap.to → interpolate
// duration → [startFrame, startFrame + duration * fps]
// delay → startFrame = delay * fps
```

**录制路径（GSAP 复杂）**：

```bash
# 使用 gsap-video-export 直接录制
npx gsap-video-export <url_or_html_file> \
  --output output.mp4 \
  --fps 30 \
  --resolution 1920x1080 \
  --codec libx264
```

##### 5A.3 通用转换规则

| 原始模式 | Remotion 替代 |
|----------|--------------|
| `requestAnimationFrame` | 删除循环，用 `useCurrentFrame()` |
| `element.style.x = v` | React `style={{ transform }}` |
| `setTimeout / setInterval` | `<Sequence from={frame}>` |
| `Math.random()` | `random('seed')` from remotion |
| `Date.now()` / `performance.now()` | `frame / fps` |
| `window.innerWidth` | `useVideoConfig().width` |
| `clock.getDelta()` (Three.js) | `frame / fps` |
| `useFrame()` (R3F) | 在 `@remotion/three` 的 ThreeCanvas 内直接用 frame |

##### 5A.4 按用户需求定制

1. 替换硬编码文案 → 用户指定 `text`
2. 替换颜色值 → 用户指定 `colors`
3. 缩放时间线 → 按 `duration_seconds * fps` 调整帧范围
4. 适配分辨率 → 调整画布和元素尺寸
5. 替换素材 → 图片/SVG/字体替换为用户资源

#### 分支 B：从零生成（无合适参考时）

使用官方模板的 AI 代码生成管线：
1. 加载 [Remotion System Prompt](https://remotion.dev/docs/ai/system-prompt) 作为 Agent 知识
2. 结合官方模板的 Skills 系统（Charts、Typography、Transitions、Spring Physics 等）
3. 生成 Remotion 组件代码
4. 使用 `remotion-animated` 简化声明式动画编写

**从零生成默认质量线**：
- 风格默认：modern-minimal
- 输出默认：至少 1440x1440、60fps；优先 120fps
- 视觉要求：避免模板感和普通演示感，必须有明确镜头中心、层次深度、至少一处高级细节（如粒子带、折射、阴影、轨道层、信号纹理、视差）
- 工程要求：所有动画保持帧确定性，可直接渲染，不依赖实时交互

---

### Step 6: 渲染输出

```bash
# 方式1：优先走仓库脚本（自动探测 entry / Composition / 浏览器）
./scripts/render.sh --comp MyComp --output out/output.{format}

# 方式2：原生 Remotion CLI 渲染（代码转换路径）
cd remix-workspace
npx remotion render src/index.ts MyComp \
  --output out/output.{format} \
  --width {width} --height {height} \
  --fps {fps} --codec {codec}

# 方式3：gsap-video-export 录制（GSAP 录制路径，需要 ffmpeg）
cd ..
./scripts/render-gsap.sh <html_file_or_url> --fps {fps} --resolution {width}x{height}
```

**codec 映射**：mp4 → `h264` / webm → `vp8` / gif → `gif`

**默认渲染要求**：
- 分辨率不得低于 1024 像素主边
- 帧率不得低于 60fps，除非用户明确接受降级
- 优先尝试 120fps；若首次渲染失败，再按 60fps 重试并说明原因
- 验证时应优先检查真实输出文件元数据，而不只看命令行日志

---

### Step 7: 失败处理

**第 1 次重试**：分析错误，修复代码（类型错误、缺少依赖、路径错误）
**第 2 次重试**：简化动画逻辑，降低复杂度
**仍失败**：
- 如果是代码转换路径失败 → 切换到 gsap-video-export 录制路径
- 如果是从零生成失败 → 降低动画复杂度重新生成
- 通知用户，提供选项

---

### Step 8: 沉淀为本地模板

渲染成功后自动保存：

```
local-templates/{type}-{style}-{timestamp}/
├── metadata.json       # 标签、来源URL、许可证、可配置参数
├── src/                # Remotion 组件源码
├── public/             # 静态资源
└── thumbnail.gif       # 前3秒渲染为 GIF 预览
```

**metadata.json**：
```json
{
  "name": "",
  "description": "",
  "tags": [],
  "created_at": "",
  "source": { "url": "", "license": "", "original_library": "" },
  "reference_search": {
    "keywords": [],
    "candidates": []
  },
  "parameters": {},
  "output": { "resolution": "", "fps": 0, "format": "" }
}
```

---

## 约束

1. **帧确定性**：所有 Remotion 动画逻辑仅依赖 `frame` 和 `fps`，不依赖时间/随机数(除 seeded random)/外部状态
2. **安全性**：从下载代码中移除 `eval()`、网络请求、文件系统操作
3. **许可证**：严格遵守白名单，metadata.json 中记录来源
4. **渲染超时**：单帧 30 秒，总渲染 10 分钟。超时自动降级
5. **不重复造轮子**：Lottie 用 @remotion/lottie，Three.js 用 @remotion/three，CSS 用 remotion-animation，优先使用现有库
6. **先检索后编码**：除非工具不可用且已明确降级，否则 Step 5 之前必须完成 Step 3 / 4 的证据输出
7. **默认风格**：用户未指定风格时，统一按 modern-minimal 处理
8. **默认质量档**：用户未指定输出规格时，默认目标是 >=1K 且 >60fps，优先 120fps

---

## 真实示例（供 Agent 照葫芦画瓢）

### 示例 1: "数据从左飞入的柱状图"

**Step 1 解析**:
```json
{
  "animation_type": "bar-chart-entrance",
  "style": "modern",
  "colors": ["#4F46E5", "#06B6D4", "#10B981", "#F59E0B", "#EF4444"],
  "text": "Q1 销量对比",
  "duration_seconds": 4,
  "resolution": "1920x1080",
  "fps": 30,
  "output_format": "mp4"
}
```

**Step 2 本地模板**: 未命中（空库）

**Step 3 检索关键词**: `["bar chart animation entrance", "d3 chart reveal gsap", "data visualization motion"]`
- 假设 GitHub 找到参考: `https://github.com/jaketrent/animated-bars` (MIT)

**Step 4 评估**: 视觉相似度 85% + 可改编性 75% + 许可证 100% + 复杂度 80% = **82% 命中**

**Step 5 改编** (写进 `remix-workspace/src/BarChart.tsx`):

```tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, Easing } from 'remotion';

const data = [
  { label: '一月', value: 120, color: '#4F46E5' },
  { label: '二月', value: 180, color: '#06B6D4' },
  { label: '三月', value: 250, color: '#10B981' },
  { label: '四月', value: 210, color: '#F59E0B' },
  { label: '五月', value: 320, color: '#EF4444' },
];

export const BarChart: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, fps * 0.5], [0, 1], {
    extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ background: '#0F172A', padding: 80, fontFamily: 'sans-serif' }}>
      <h1 style={{ color: 'white', fontSize: 64, opacity: titleOpacity }}>Q1 销量对比</h1>
      <div style={{ display: 'flex', alignItems: 'flex-end', gap: 40, height: height * 0.6, marginTop: 80 }}>
        {data.map((d, i) => {
          const startFrame = Math.round(fps * 0.3) + i * Math.round(fps * 0.15);
          const endFrame = startFrame + Math.round(fps * 0.8);
          const barHeight = interpolate(frame, [startFrame, endFrame], [0, d.value * 2], {
            easing: Easing.out(Easing.cubic),
            extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
          });
          return (
            <div key={d.label} style={{ flex: 1, display: 'flex', flexDirection: 'column', alignItems: 'center' }}>
              <div style={{ width: '100%', height: barHeight, background: d.color, borderRadius: 8 }} />
              <div style={{ color: 'white', marginTop: 16, fontSize: 28 }}>{d.label}</div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

**Step 6 渲染**:
```bash
./scripts/render.sh --comp BarChart --output out/q1-bars.mp4 --duration 120
```

**Step 8 沉淀**:
```bash
./scripts/save-template.sh bar-chart-modern '现代风格柱状图入场动画' 'chart,bar,entrance' 'https://github.com/jaketrent/animated-bars' MIT d3
```

### 示例 2: 复杂 GSAP 动画（走录制路径）

如果用户给一段 GSAP 链接 `https://codepen.io/anon/pen/xxxxx` 且包含 ScrollTrigger，按 `rules/gsap-to-remotion.md` 的"复杂度评估清单"判定走录制路径：

```bash
./scripts/render-gsap.sh https://codepen.io/anon/pen/xxxxx --fps 30 --resolution 1920x1080
```

---

## 排错指南 (Troubleshooting)

| 症状 | 原因 | 修复 |
|------|------|------|
| `create-vibe-motion` 拉取失败 | 远程脚手架不可用 | `init-workspace.sh` 会优先复制仓库自带模板；只有缺模板时才回退到远程脚手架 |
| `npx remotion render` 报 `Cannot find Chrome` | Chromium 未安装或路径不对 | 重跑 `setup-linux.sh` 的 Step 4；或将 `PUPPETEER_EXECUTABLE_PATH` 指向系统 Chromium 或 `~/.local/chrome/chrome-headless-shell-linux64/chrome-headless-shell` |
| 渲染卡在单帧超过 30 秒 | 动画里有 `Math.random()` 或外部 IO | 改成 `random('seed')`；删除 `fetch`/`fs` 调用 |
| 编译报 `Cannot find module 'remotion-animated'` | 依赖没装 | `cd remix-workspace && pnpm add remotion-animated remotion-animation @remotion/three @remotion/lottie three @react-three/fiber` |
| `gsap-video-export` 报 `puppeteer` 错误 | 沙箱里没法跑 Chromium | 走代码转换路径（见 `rules/gsap-to-remotion.md`），或设置 `PUPPETEER_EXECUTABLE_PATH` |
| Composition 找不到 | 组件 ID 与 `Composition` 标签的 `id` 不一致 | `render.sh --comp` 改成 `Root.tsx` 里的 `<Composition id="...">` |
| 第一次 `pnpm install` 报 `EACCES` | 用 `sudo` 装过全局 npm 包 | 删除 `~/.npm` 下的 root 文件，或用 nvm 重新装 Node |
| 输出 GIF 文件巨大 | 默认 30fps 全帧 | 改用 `--fps 10`，或 `ffmpeg -i out.mp4 -vf "fps=10" out.gif` |

---

## 与同类 Skill 的差异

| 维度 | 通用 remotion-ai skill | remix-animation（本） |
|------|----------------------|---------------------|
| 动画来源 | LLM 凭空生成 | 社区参考 + LLM 改编（更稳定） |
| 外部 API | 通常需要 OpenAI | **不需要** |
| 模板沉淀 | 无 | 本地 `local-templates/` 累计 |
| GSAP 复杂动画 | 不支持 | 录制路径兜底 |
| 离线运行 | 不行 | 检索步骤可降级到"从零生成" |
| 沙箱友好度 | 一般 | 全部命令都用 npm/pnpm，可控可重试 |

---
> Source: [embalmer-Y/remix-animation](https://github.com/embalmer-Y/remix-animation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
