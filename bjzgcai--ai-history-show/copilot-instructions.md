## ai-history-show

> - **技术栈**: HTML5 + CSS3 + Vanilla JS + Three.js（3D地球）

# AI-History-Show 项目指南

## 项目概述

交互式AI历史展览大屏应用。

- **技术栈**: HTML5 + CSS3 + Vanilla JS + Three.js（3D地球）
- **主入口**: `index.html`（Three.js地球 + 里程碑展示）
- **数据文件**: `milestones-data.js`（由 `manage/generate.js` 生成，勿手动编辑）

## 文件结构

```
AI-History-Show/
├── index.html                   # 主展示页
├── milestones-data.js           # 生成的数据（勿手动编辑）
├── milestones-data-full.js      # 原始手工数据（仅供参考）
├── resources/
│   ├── images/                  # 里程碑图片（23个文件夹，~35MB，append-only）
│   └── videos/                  # 视频元数据 JSON（18个文件，append-only）
├── manage/
│   ├── catalog.js               # ✏️ 分类与展示顺序配置
│   ├── events.js                # ✏️ 各事件内容
│   ├── generate.js              # 生成脚本
│   ├── server.js                # 管理后台服务
│   └── admin.html               # 管理界面
├── DEPLOYMENT.md                # 部署指南
└── AGENTS.md                    # 本文档
```

## 内容管理工作流

**只需编辑两个文件**，然后运行生成脚本：

```bash
# 编辑 manage/catalog.js（分类/顺序）或 manage/events.js（事件内容）
node manage/generate.js
# → 重新生成 milestones-data.js
```

### 数据结构（events.js）

每个事件 key（如 `"1956-dartmouth"`）包含：

| 字段 | 说明 |
|------|------|
| `year`, `title` | 年份、标题 |
| `location` | `{name, country, coordinates: [lat, lng]}` |
| `description` | 正文描述 |
| `figures` | `[{name, role}]` |
| `quoteText` / `quotePage` | 引用文字（`\n` → `<br>`）/ 出处 |
| `commentaryVideo` | 解说视频 mp4 URL |
| `images` | 相对路径数组 |
| `videos` | `[{id, title, channel, duration}]`，YouTube 视频 |

### AI100 成就新增 schema（必须遵守）

以后新增 BenchCouncil AI100 achievement 时，必须一次性满足以下内容与网页布局要求，避免后续反复返工：

#### 1. 顶部三联视觉区

AI100 页面顶部必须像旧 achievement 一样有 3 个资料卡片：

  1. `images[0]` 为相关科学家 / 人物 / 团队 / 机构照片。
     - 优先真人 portrait；如果没有可靠 portrait，可用团队、机构、实验室或历史照片。
     - 必须在 `imageMeta[images[0]]` 写清 caption、subcaption、source、sourceUrl、license/usage。
  2. `images[1]` 为成就本身的 visualization / architecture / algorithm explainer。
     - 不能放第二张人物照。
     - 优先本地原创 SVG/PNG explainer；不要直接复制受版权保护的论文图。
  3. `achievement.visualModules[0]` 为相关文章、论文、项目页或档案页卡片（`type: "archiveLink"`）。
     - 必须包含 `site/title/description/url/source/action`，并推荐包含 `license/usage`。
     - 这是右侧 article/source 卡片，不是普通图片。
     - `site/title/description/license/action/usage` 都必须支持 `{en, zh}`；中文页不能出现 `Open project page`、`Reference link only`、`downloadable implementation materials`、`paper record` 这类英文 UI 句子。
     - `zh` 字段必须写成中文用户能直接理解的自然句，例如 `打开项目页面`、`仅作为参考链接；本地图片为重绘，不复制出版方图形。`、`弗赖堡团队的项目页面，提供 U-Net 实现材料下载。`
     - 允许保留通用专名、论文标题、代码仓库名、缩写和模型名（如 GitHub、arXiv、U-Net、Faster R-CNN、DOI），但周围解释文字必须是中文。

#### 2. 底部互动解释区

底部必须是旧 achievement 那种 paper-demo / visual-demo 布局：

- `achievement.visual` 必须映射到非 generic 的 demo renderer。
- 左侧为大 visual/demo，用于展示算法流程、架构、数据流、系统路径或成果可视化。
- 右侧为两个说明盒：
  - 第一个盒子说明文献线索、架构线索、历史线索、实验线索或专家线索。
  - 第二个盒子必须是 `Interaction point` / `互动点`，说明观众可以怎样交互理解该成就。
- 如果没有现成 visual renderer，必须新增 renderer 或使用 `buildImagePaperDemo` / `buildPaperDemo` 风格实现，不要让页面退回到只有一句文字的 generic demo。

#### 3. 右侧文字栏

- `commentarySections` 至少包含 `Historical Background`、`Core Idea`、`Long-Term Legacy`。
- 以上三个 context sections 的英文和中文正文都必须至少 2 个完整句子；不要只写一句定义或一句影响概述。
- 第一句应交代历史/技术背景或问题，第二句应说明为什么这个成就重要、如何工作、或它和相邻 AI 成就的关系。
- `Long-Term Legacy / 长期影响` 必须说明专家如何评价该成就，不能只写普通影响描述。
- 推荐包含类似句式：`Experts generally treat...` / `专家通常把/认为...`。
- 人物/portrait 卡片的 `imageMeta.subcaption` 仍应保持短句，只说明人物与成就的关系（如 `Isomap 主要作者`），不要把这里当作 context 段落。

#### 4. 资料来源

- `achievement.sources` 至少 3 条，推荐 4 条，风格参考旧 achievement。
- 必须包含主论文 / 原始资料。
- 还应补充背景、项目页、人物资料、机构资料、代码、图片来源、历史回顾或官方出版页面等，按该成就实际情况选择。
- 每个 source 必须有 `type`、`label`、`url`，并提供真实 `{en, zh}` 双语文字。
- 不要只放一条论文链接。

#### 5. Quiz

- 每个新增 AI100 achievement 必须同步在 `manage/quizzes.js` 添加 quiz。
- Quiz 布局必须匹配旧 achievement 的浏览检查点：左侧相关材料，右侧快速挑战，包含 4 个选项。
- 题目必须简单、清楚，适合普通观众，不要考冷门细节。
- 选项源数据可以保持固定顺序；前端会随机显示答案顺序。
- Quiz 相关材料必须有图片或资料，不要缺图、缺信息。

#### 6. 双语与页面语言

- 所有页面可见字段必须有真实 `{en, zh}` 内容。
- 中文页显示中文，英文页显示英文。
- 不要把英文直接复制到 `zh` 字段，除非是通用专名、缩写或模型名（如 ReLU、LeNet、AlexNet、arXiv）。
- 需要覆盖：`title/description/location/figures/commentarySections/achievement/imageMeta/visualModules/sources/quizzes`。
- 人名在中文页必须用中文译名或中文可读形式：例如 `Kaiming He` 应显示为 `何恺明`，`Shaoqing Ren` 应显示为 `任少卿`。仓库路径、论文标题或机构英文专名可保留，但人物显示名和人物说明不能只用英文。
- `imageMeta.caption/subcaption/sourceName/license/usage` 必须双语；portrait 卡片的中文 caption 应类似 `何恺明肖像`，subcaption 应是一句短关系说明，如 `Faster R-CNN 共同作者`，不要写长段落。
- `achievement.visualModules` 右侧卡片也属于页面可见字段，必须完整本地化 `site/title/description/license/action/usage`。不要出现中英混杂句子，如 `项目 page and downloadable implementation materials from the 弗赖堡 group`。
- 地区索引 / region picker 在中文页必须显示中文，且只显示国家级地区，不显示城市组合，如 `Tokyo, Japan`、`Fukuoka and San Diego`、`Zurich, Switzerland`。地区筛选应使用国家（如 `日本`、`美国`、`瑞士`），不要把城市字符串当作可选地区。
- 中文来源标签也要自然本地化，例如 `ACM Digital Library` → `ACM 数字图书馆`，`MIT Press` → `麻省理工学院出版社`，`Open DOI page` → `打开 DOI 页面`。

#### 7. 生成与验证

- 修改 source 后必须运行 `node manage/generate.js`。
- 至少运行 `npm run lint` 和 `npm test`。
- 新增或大批修改 AI100 achievement 时，运行 `npm run validate:ai100-context`，确保主要 context sections 满足至少两句要求。
- 新增 archive/right-side cards 后，应抽查生成后的 `milestones-data.js`，确认 `visualModules` 的中文字段没有英文 UI 句子残留。
- 若影响启动或页面加载，运行 `npm run validate:startup`。
- 不要手动编辑 `milestones-data.js`；它由生成脚本输出。

### 当前事件（5 分类，21 个）

- **AI创世纪**: 1956-dartmouth, 1957-perceptron, 1969-ai-winter
- **神经网络复兴**: 1986-backpropagation, 1989-cnn, 1986-rnn, 1997-lstm
- **深度学习**: 2012-alexnet, 2014-highway-network, 2015-resnet, 2016-densenet, 2014-gan, 2014-attention, 2017-transformer
- **大模型时代**: 2018-bert, 2018-gpt, 2023-agents, 2025-llm-competition
- **AI for Science**: 2020-alphafold, 2019-ai-feynman, 2024-ai-scientist

## 技术细节

### Three.js 地球

- 相机绕地球飞行（`camCurrent`/`camTarget`），地球本身不旋转
- 页面切换时相机平滑飞向目标坐标，红色标记点闪烁后常亮
- 坐标 `[0, 0]` 为无效占位符，跳过不触发飞行

```javascript
// 经纬度 → 相机角度
phi   = (90 - lat) * (Math.PI / 180)
theta = -lng * (Math.PI / 180)
```

### 管理后台（manage/server.js）

- `GET /api/generate/diff` — 预览变更（对比 manage/ 与已生成的 milestones-data.js）
- `POST /api/generate` — 执行生成
- `POST /api/events` — 保存 events.js（写入前同步 YouTube 视频到 resources/videos/{key}.json）

### 注意事项

- VS Code TS 语言服务对中文引号在模板字符串中报误报，Node.js 实际运行正常
- `resources/` 目录为 append-only，不删除已有文件
- 生成的里程碑 ID 格式：`milestone-{key}`（如 `milestone-1956-dartmouth`）

---
> Source: [bjzgcai/AI-History-Show](https://github.com/bjzgcai/AI-History-Show) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
