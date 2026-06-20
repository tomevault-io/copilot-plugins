## hyperframes-video-skill

> Turn a project/tool/engineering story into a short, vivid product video for an everyday-developer audience and help publish it to Bilibili. Story-driven scriptwriting, voice-cloning voiceover (MMX/MiniMax, falls back to system TTS), HyperFrames + GSAP animation (DrawSVG/MotionPath/kinetic captions), cover + metadata, browser-assisted upload, collection management.


# Bilibili Product Video

Use this skill when the user wants to turn a project, tool, demo, or engineering story into a short video for a general/developer audience, especially when they mention product introduction, B站投稿, 合集, 封面, 配音, 音色克隆, MMX, HyperFrames, or using their own voice.

> **自更新（每次使用前先做）**：本 skill 由 git 仓库管理。开始任何工作前先拉最新：`bash "$SKILL_DIR/scripts/update.sh"`（等价于 `git -C "$SKILL_DIR" pull --ff-only --quiet`）。`$SKILL_DIR` = 本 SKILL.md 所在目录。claude 与 codex 两端通过 symlink 指向同一 git 克隆，拉一次两端同步。
>
> **开源依赖**：HyperFrames（Apache-2.0，`npx hyperframes`）做渲染。MMX 是私有音色克隆 CLI，**可选**；没有它时配音自动回退系统 TTS（见 §4）。

## Default Goal

Produce a complete first-pass video package and help the user publish it:

- Rendered video file.
- Cover image.
- Voiceover audio and subtitles.
- Bilibili title, description, tags, and collection plan.
- Browser-assisted upload, with explicit user confirmation before irreversible publishing actions unless the user already clearly authorized them.

## Inputs To Collect

Prefer using existing context before asking questions. Only ask if the missing detail changes the output materially.

- Project name, repo path, public URL, and open-source URL.
- Audience tier (默认 **普通大众程序员**): 他们知道 AI / Claude / 终端是什么、自己会写代码，但**不知道你项目的内部架构**（Adapter、driver 接口、协议、ASR/TTS 链路）。讲东西用「结果 + 类比」，不讲内部行话。比"非专业大众"高一档、比"硬核极客"低一档。
- Core scenario: the real pain point, who had the problem, and why the tool matters.
- Voice (音色克隆): 用 `scripts/voice.sh` 生成配音——配了 MMX/MiniMax 克隆声就用克隆声（声音 ID 走环境变量 `MMX_VOICE_ID`），没配就**回退系统 TTS**（macOS `say` / Linux `espeak`）。见下方「Generate Voiceover」。
- Preferred 合集/系列 name (可选): 同一系列的视频归到一个合集；名字由用户提供，无则不强求。
- Episode format: 常规故事 / 对比评测 / 翻车踩坑（见下方「形态轮换」）。

## 创作标准（核心 — 2026-06 升级，优先于下方模板）

来自前几期的复盘：早期片子**内容单调（每个场景同布局）、脚本是功能罗列、太钻（讲内部架构）、动画不形象**。以下四条是新标准，每期必守。标杆参照：B站「工科男孙老师」（故事壳 + 真实物当锚 + 逐段高亮 + 信号流框图 + 章节条 + 人格化吐槽）、「爱上半导体」（自绘概念动画把抽象变直观 + 大字幕 + "X vs Y 区别"对比形态，单条 98 万播放）。

### ① 受众下沉：删内部行话
- 目标受众 = **普通大众程序员**（见 Inputs）。**禁止**出现这些内部词：Adapter、driver 接口、router.Register、active_driver、协议名、ASR/TTS 链路、session 等。
- 一律换成**结果或类比**：不说「driver matrix 热切」，说「想换哪个 AI 就换哪个」；不说「本地 ASR/TTS」，说「连你说的话都不出这台电脑」。
- curl/终端可以露（程序员爱看真东西），但**作为"证据一闪"**，不逐字段讲解。

### ② 故事化脚本骨架（默认形态）
不要功能罗列。每期一条**故事弧**，且有"我"这个角色和真实赌注：
```
钩子(5s 痛点/反差) → 我为什么做这个(我遇到的具体麻烦) → 试 / 翻车(过程有波折) → 啊哈时刻(它怎么解决) → 一句话怎么用 → 开源/CTA
```
- 钩子库（挑一个，不平铺）：「我受够了X，于是…」「都说Y很难，其实…」「同事/老板说Z，我顺手做了…」「有手机/有现成的，为什么还要这个？」
- 多用第一人称、具体场景、有情绪起伏；**少用"它支持…它可以…"的清单句**。

### ③ 形态轮换（破单调）
不要每期都同一种片子。按节奏轮换：
| 形态 | 何时用 | 公式 |
|---|---|---|
| 常规故事 | 默认 | 上面的故事弧 |
| **对比评测** | 每 3 期插 1 期 | 「X vs Y」「为什么用X不用Y」——云AI vs 本地AI、智能助手 vs 自己搓、一堆工具 vs 一个入口。对比形态天然高播放。 |
| 翻车踩坑 | 有真实事故时 | 「我把X搞炸了」→ 复盘 → 修好。真实、好笑、共鸣。 |

### ④ 形象动画 + 布局轮换（破抽象、破单调）
- **禁止**整片都用「左文字栈 + 右面板」同一布局。每 2-3 个场景**换一种构图**：满屏大字、居中单图、左右对调、上下分层、纯动画特写。
- 至少有**一个高潮 demo beat**（信息密度最高、最出彩的一幕），别让每幕权重一样。
- **隐喻优先于文字面板**——把抽象概念变成能看见的东西。逐期积累「隐喻库」：
  | 概念 | 形象隐喻 |
  |---|---|
  | AI 后端可切换 | 可插拔**卡带 / 转接头**，咔哒换一个 |
  | 本地 vs 云 | 数据**出不出家门**（云=飞出窗外，本地=锁在屋里）|
  | 厂商锁死 | **上锁的笼子 / 焊死的盖子** |
  | 适配层 | **万能转接头 / 翻译官** |
  | 自绘概念图 | 像爱上半导体那样让图表/连线**自己画出来**，配大字幕 |
- **隐喻生成法（每期现造，别照搬旧图）**——抽象概念卡壳时按这三步走，比查表更可靠（借鉴自 [ian-xiaohei-illustrations](https://github.com/helloianneo/ian-xiaohei-illustrations) 的方法论，MIT）：
  1. **概念→物理动作**：卡住 / 漏掉 / 变重 / 分拣 / 沉淀 / 发酵 / 开门 / 折叠 / 拆包 / 回流。
  2. **结构→低科技物件**：坏机器 / 纸箱 / 抽屉 / 漏斗 / 秤 / 邮筒 / 门 / 井 / 梯子 / 水管 / 线团 / 闸门 / 转盘 / 黑盒 / 压面机。一次只用 1-2 个，别堆满。
  3. **让角色承担动作**：不是站旁边解说，而是卡在机器里、拉错线、守门、搬运、修补、称重——动作服务核心意思，别为怪而怪。
- **固定 IP 角色（系列连续性）**——指定一个贯穿全系列的吉祥物（默认沿用上面隐喻法里的执行者），每期都让它做核心动作而非装饰；表情克制、有点呆、不卖萌。用 SVG/CSS 自绘成可被 GSAP 驱动的矢量件（这样能 DrawSVG 自绘 / MotionPath 运动），对标「工科男孙老师」的人格化吐槽，给观众记忆锚点。具体形象一旦定稿写进本节固化。
- **构图模式库（配合「每 2-3 幕换构图」）**——一幕只用一种结构，别混：**Workflow**（左输入→中处理→右输出，橙箭头主流向）/ **系统局部**（只画 3-5 个核心模块，角色参与其一）/ **前后对比**（左乱右稳，中间箭头）/ **角色状态**（2-4 个小状态各配短标注）/ **概念隐喻**（一个大怪物件，少量输入一个输出）/ **方法分层**（一层层盒子，角色在旁搭建）/ **地图路线**（一条弯路径少量节点）/ **小漫画分镜**（2-4 格，每格一个动作）。
- 已验证好用的视觉件（沿用）：底部**章节进度条**（随播放高亮，留存利器）、**动画信号流**（节点间流动小点 + active 项发光）、真机实拍当证据、点阵/Nothing 风格。

## Workflow

### 1. Understand The Product

Read the project README, docs, app screenshots, and public site when available. Extract:

- One-sentence plain-language value proposition.
- The before/after story.
- The key demo moments worth showing.
- Terms that should be simplified for non-specialists.

For Agent Room-like projects, frame it as:

- A tool made to solve coworkers' Windows environment/software installation troubleshooting.
- Slave mode lets a user's machine without local AI be coordinated by the creator's AI.
- Multiple AI agents can enter a room and talk through a problem like a meeting or voice room.
- The hook is practical: "同事电脑没 AI，也能让我的 AI 帮他排查问题".

### 2. Create A Video Folder

Create a dedicated folder under `videos/`:

```text
videos/episode-NN-topic-bilibili/
├── assets/
├── preview-frames/
├── design.md
├── narration.md
├── narration.txt
├── index.html
├── publish-draft.md
├── cover-*.png
└── *.mp4
```

Keep generated assets scoped to this folder. Do not disturb unrelated user changes in the repo.

### 3. Write The Script

大白话、短句、前 5 秒钩子。**按「创作标准 ②」的故事弧写，不要功能罗列**。守「创作标准 ①」删内部行话。时长 60–120 秒（程序员向可放宽到 2–3 分钟但不注水）。

写完自检：
- [ ] 有"我"和真实赌注吗？还是在念说明书？
- [ ] 出现内部行话了吗（Adapter/driver/协议…）？换成结果/类比。
- [ ] 钩子是"痛点/反差"还是"功能介绍"？
- [ ] 有过程波折（试/翻车）还是一路顺？
- [ ] 句子是"我遇到…我做了…"还是"它支持…它可以…"？后者要改。

正面例（故事感、第一人称、有波折）：

```text
我一直想要个能说话的 AI，但市面上的智能音箱我一个都看不上——
背后接哪个 AI 根本不归你管，全锁死在人家云上。
不服，我自己搓了一个。
第一版翻车了，声音卡得像复读机……
改到第三版才顺：现在我想换哪个 AI 就换哪个，断了网也能聊。
```

反面例（功能罗列，**别这么写**）：

```text
它支持六个 AI 后端。它可以热切换。它的驱动层负责适配。它还能本地离线。
```

### 4. Generate Voiceover（音色克隆，带回退）

统一用本仓库的 `scripts/voice.sh` 生成配音——它会**自动探测**：

```bash
bash "$SKILL_DIR/scripts/voice.sh" narration.txt assets/mikas-voice.mp3
```

- **配了 MMX/MiniMax**（`mmx` 在 PATH 且设了 `MMX_VOICE_ID`）→ 用**克隆声**：
  `mmx speech synthesize --voice "$MMX_VOICE_ID" --emotion happy --speed 1.12 --subtitles --language Chinese --out <out>`
- **没配** → **回退系统 TTS**：macOS `say` / Linux `espeak`(-ng)，转成 mp3。保证任何人 clone 下来都能出片。
- 可选环境变量：`MMX_VOICE_ID`（克隆声 ID）、`VOICE_EMOTION`（默认 happy）、`VOICE_SPEED`（默认 1.12）。

产物：`assets/<name>.mp3` + `assets/<name>.srt`（MMX 带 `--subtitles`；回退路径用 `hyperframes transcribe` 或按句估时生成 SRT）。

**语调/语速（重要 — 默认平淡偏慢，必须调）**：默认 emotion=neutral 会显得平。默认 **happy + speed 1.12**（活泼带劲）。改了 speed/emotion 后**时长会变**，必须重新拿 SRT 对齐场景/字幕/章节。

> 说明：MMX 是私有的 MiniMax 包装 CLI（音色克隆），非开源依赖；没有它时本 skill 自动走系统 TTS，不影响主流程。

After generation, verify duration and listen/inspect enough to catch obvious failures.

### 5. Build The HyperFrames Composition

Use the `hyperframes` and `hyperframes-cli` skills when creating or rendering the video.

Composition guidance:

- 1920x1080, 30fps by default.
- **守「创作标准 ④」**：布局每 2-3 幕轮换，不许整片同一个「左文字+右面板」；隐喻优先于文字面板；至少一个高潮 demo beat。
- 已验证好用、直接沿用的件：底部**章节进度条**（onUpdate 里按时间高亮当前章）、**动画信号流**（节点 + 流动小点 packet + active 发光，用有限 repeat 别用 `repeat:-1`，否则 totalDuration=Infinity 破坏抽帧）、真机实拍当证据、点阵风格。
- 用真截图/真终端输出当"证据"，但只一闪，别逐字段讲。
- Use large captions synced to the narration（字幕数组 `[start,end,text]` 拆成短句卡，跟 SRT 对齐）。
- 渲染前先 `npx hyperframes snapshot --at t1,t2,...` 抽关键帧，读 contact-sheet.jpg 肉眼检查；改完时长记得改 `data-duration` 和 GSAP grain tween 的 duration。
- Export preview frames at several timestamps before final render.

Run the normal validation loop:

```bash
npx hyperframes lint
npx hyperframes inspect
npx hyperframes render
```

**片头/片尾拼接 + 配音合成（音频管线，已踩坑）**：

- **配音合进正片**：`ffmpeg -i video.mp4 -i voice.mp3 -c:v copy -c:a aac -ar 44100 -ac 1 -map 0:v:0 -map 1:a:0 out.mp4`。配音用 `scripts/voice.sh`（默认 44100Hz，见下）。
- **拼接 intro+正片+outro：三段音频必须同采样率、零重采样**，否则会出**杂音**。最稳：配音出成 44.1kHz/单声道（`voice.sh` 已默认 `VOICE_SAMPLE_RATE=44100`；MiniMax 支持 44100、**不支持 48000**），片头/片尾的静音轨也用 44.1kHz/单声道，concat 不加任何 `aformat/aresample` 升采样：

  ```bash
  ffmpeg -y -i intro.mp4 -i ep-voiced.mp4 -i outro.mp4 \
    -f lavfi -i anullsrc=channel_layout=mono:sample_rate=44100 \
    -filter_complex "[0:v]scale=1920:1080,fps=30,setsar=1[v0];[1:v]scale=1920:1080,fps=30,setsar=1[v1];[2:v]scale=1920:1080,fps=30,setsar=1[v2];[3:a]asplit=2[sa][sc];[sa]atrim=0:<intro_dur>,asetpts=N/SR/TB[a0];[1:a]asetpts=N/SR/TB[a1];[sc]atrim=0:<outro_dur>,asetpts=N/SR/TB[a2];[v0][a0][v1][a1][v2][a2]concat=n=3:v=1:a=1[v][a]" \
    -map "[v]" -map "[a]" -c:v libx264 -crf 19 -pix_fmt yuv420p -c:a aac -ar 44100 -ac 1 "$OUT"
  ```
  自检：`ffmpeg -i out.mp4 -af volumedetect -f null -` 的 `max_volume` 不应是 0.0dB（削峰=杂音线索）。
- **重出 TTS 时长会变**（MMX 节奏有随机性，同文本同语速也可能差 1s）——重出后必须按新音频重新对齐场景时间轴并改 `data-duration`，否则结尾被切。

If a command differs for the local project, follow the local HyperFrames files and CLI help.

#### 动画工具箱（2026-06 升级 — 把"形象动画"做到爱上半导体那一档）

HyperFrames + 基础 GSAP 之外，这些都**免费、可加载**，按需用来升级动画（别再只用 CSS 滑入 + 文字面板）：

1. **GSAP 高级插件（已全部免费，加一行 CDN 即可）**——做"形象动画"的主力：
   - `DrawSVGPlugin`——**让线条/图表/连线自己画出来**（概念图自绘，爱上半导体同款）。
   - `MotionPathPlugin`——元素（数据包/小点）**沿真实 SVG 路径运动**，比 x/left 平移自然得多。
   - `MorphSVGPlugin`——形状变形过渡。
   - CDN：`https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/<Plugin>.min.js`，用前 `gsap.registerPlugin(DrawSVGPlugin, MotionPathPlugin)`；`motionPath:{ path:"M..." }`、`drawSVG:"0% 100%"`。保持 loop 有限。
2. **HyperFrames 组件库（`npx hyperframes catalog` 浏览，`npx hyperframes add <name>` 拉进工程）**——113 个 block/component，重点：
   - 字幕升级：`caption-kinetic-slam`（砸字）/`caption-karaoke`/`caption-neon-glow`/`caption-particle-burst`/`morph-text`——替代手写纯字幕条。
   - 概念图：`flowchart`、`data-chart`（动画图表/流程图，做自绘概念图省事）。
   - 转场/质感：`shimmer-sweep`、`grid-pixelate-wipe`、`motion-blur`、`grain-overlay`、`vignette`。
3. **作者参考技能（`npx hyperframes skills` 装到 `.agents/skills/`，写之前 Read 对应的）**：`gsap`(含 DrawSVG/MotionPath 用法)、`lottie`(嵌 After Effects 矢量动画)、`animejs`、`css-animations`、`three`(3D)、`waapi`、`hyperframes/references/techniques.md`(技法手册 + "按内容类型选动画"对照表)。
4. **静态插画素材（可选）**：需要角色/场景插画时，可用任意文生图（如本地/云端 image-gen provider）生成元素，再用 GSAP 驱动；**纯白手绘风的 PNG 只适合插画特写/转场**，且要先确认和整片暗色调能否和谐，正片主体优先用可被 GSAP 自绘/运动的 SVG。无则用 SVG/CSS 形状 + emoji 自绘。固定 IP 角色见创作标准 ④。

选型原则：能用隐喻/自绘说清的概念，优先 DrawSVG + MotionPath；纯文字强调用 kinetic 字幕组件；别为炫技堆插件，服务于"把抽象讲形象"。

### 6. Prepare Bilibili Metadata

Write `publish-draft.md` with:

- Title options.
- Final recommended title.
- Description.
- Tags.
- Collection name.
- Pinned comment idea.
- Asset paths.

Title rules:

- Put the viewer pain point first.
- Mention the surprising capability.
- Keep it understandable without knowing the repo.

Strong title patterns:

```text
同事电脑没 AI，也能让我的 AI 远程排查？我做了个 Agent Room
我做了个 AI 排查房间：同事 Windows 装不上软件，也能拉 AI 一起会诊
让 AI 像开会一样排查问题：我下午做了个 Agent Room
```

Default description template:

```text
这期分享一个我下午做出来的 AI 工程工具：Agent Room。

它解决的是一个很真实的问题：同事 Windows 环境装软件、跑项目出问题时，来回问截图和日志太慢。

Agent Room 像一个给人和 AI 开的排查房间：
- 支持 slave 模式，用户电脑没有 AI 也能接进来；
- 我的 AI 可以调用对方环境，一起排查安装和运行问题；
- 也可以让多个 AI agent 在同一个房间里对话、讨论、处理问题。

体验地址：https://agent-room.daboluo.cc/
开源地址：https://github.com/zhoushoujianwork/agent_room
```

Default tags:

```text
AI, Agent, Windows, 开源项目, 远程协作, AI工具
```

If Bilibili rejects a tag, replace it with a nearby valid tag rather than getting stuck.

### 7. Browser Upload And Publishing

Use the Browser/in-app browser skill for Bilibili creator center work when available.

Important browser rules:

- Treat final publish, metadata save, collection creation, file upload, and deletion as external side effects.
- If the user already explicitly requested the action, proceed; otherwise confirm at action time.
- Prefer the in-app browser for stable DOM control.
- If macOS file chooser focus is unreliable, ask the user to manually choose the exact local file path.
- Do not expose cookies, tokens, or private account data.

Upload checklist:

1. Upload rendered `.mp4`.
2. Set declaration if AI-generated content is involved（创作声明 → 选「含AI生成内容」）.
3. Fill title, description, tags, category, and cover.
4. Confirm the preview and required fields.
5. Ask or proceed according to user authorization before final publish.
6. After publish, verify the video appears in video management.

B站投稿页（chrome-devtools）已踩过的坑：
- **文件被拦**：upload_file 只允许工作区根目录内的路径——成片在 `github/<repo>` 下会被拒，先拷到当前项目目录再传。
- **视频上传**：直接塞 `<input type=file>` 不触发 B站 JS；用 upload_file 指向**可见的「上传视频」按钮**，让它走 B站 自己的文件选择流程。
- **标题**：用 fill 会**追加**到原值后面；用 React 原生 setter + `input` 事件清空重设。
- **简介**：是 Quill 编辑器，execCommand/paste 都不稳；拿实例 `container.__quill.setText(text,'user')` 最可靠（多段换行也对）。
- **封面**：勾「双比例同步改动」再上传，4:3 首页推荐 + 16:9 个人空间同步成自定义封面（自动封面用我视频帧+花字，不如自己设计的）。
- **残留分P名**：单 P 投稿时左侧卡片可能残留旧标题，但**公开标题以「标题」输入框为准**，卡片名不对外、且只读不可编辑——核对输入框即可，别纠结卡片。
- **发布后核对**：去 `upload-manager/article` 确认公开标题/封面/合集都落对。

#### CLI 快传（可选；⚠ B站当前封禁第三方提交，最终仍回退浏览器）

`scripts/bili-upload.sh` 包了 [`bilitool`](https://github.com/timerring/bilitool)（维护中的 B站投稿 CLI）：

```bash
BILIUP_COOKIE=/path/to/cookies.json \
  bash "$SKILL_DIR/scripts/bili-upload.sh" \
  videos/<ep>/renders/<full>.mp4 videos/_cover-<ep>/cover.png \
  "<标题>" 174 "开源硬件,ESP32,嵌入式,..." videos/<ep>/desc.txt
```

- **现状（实测 2026-06，重要）**：biliup / bilitool **都能把视频字节+封面成功传上 B站**，但最后「创建稿件」被 B站拒——`biliup: 21150 投稿入口升级中` / `bilitool: 投稿工具已停用`。换第三方工具同样被拦，这是 B站策略不是工具 bug。**所以最终投稿务必走上面的浏览器创作中心**；本脚本用于快速校验登录/上传链路，及 B站若恢复接口即可直接用。脚本检测到拦截会自动提示回退浏览器并退出码 2。
- **登录免扫码（cookie 复用）**：`biliup login` 生成的 `cookies.json`（顶层 `cookie_info` + `token_info.access_token`）可转成 bilitool 的 `-f` 格式（`{"data":{"access_token":..., "cookie_info":...}}`，cookie 顺序须 SESSDATA/bili_jct/DedeUserID/__ckMd5/sid）——脚本里已内置转换，设 `BILIUP_COOKIE` 即自动登录。
- **file_upload 工具传不了本地成片**：只接受「对话附件」，仓库路径一律被拒；桌面控制对浏览器是只读等级、点不了原生文件框。所以浏览器投稿时**视频/封面那一下需用户手动拖拽**，其余元数据再交给浏览器自动填。
- 不要提交 `cookies.json` / bilitool 登录态到仓库（含账号凭据）。

### 8. Collection Management

If the user asks for a collection, create or reuse:

合集名由用户提供（同一系列归一个合集）。无则可建议一个贴合内容的名字，例如：

```text
<你的系列名>，例：「程序员的 AI 工程工具」「XX 研发实录」
```

Collection cover is usually required. Use the current episode cover. If browser automation cannot drive the system file picker, stop and ask the user to select the exact cover file manually.

After collection creation, add the published video to the collection when Bilibili allows it.

### 9. Post-Publish Fixes

Immediately check video management after publishing:

- If the title fell back to a filename, edit it to the recommended title.
- Verify description and tags survived.
- Add to the collection.
- Capture the BV URL for the user.

When changing a published video's metadata, proceed if the user already requested the fix. Otherwise ask before saving.

## Quality Bar

The final package is not done until:

- Video renders successfully and has audio.
- Cover exists and matches the topic.
- Script is understandable by a non-specialist.
- Bilibili metadata is ready and not just a copied filename.
- Publish status or remaining manual step is clearly reported.

新增红线（2026-06，对应四条创作标准，未过不算完成）：

- **不堆内部行话**：脚本/字幕/标题里没有 Adapter、driver、协议、ASR/TTS、session 等内部词（受众=普通大众程序员）。
- **不是功能罗列**：脚本有故事弧 + "我" + 波折，不是"它支持…它可以…"清单。
- **不单调**：布局至少轮换 2-3 种，不是整片同一个左文字+右面板；有一个高潮 demo beat。
- **够形象**：抽象概念用了隐喻/自绘动画，而不是纯文字面板。
- **配音不平**：用了 emotion + speed（默认 happy/1.12），听过确认有起伏、不拖。
- **标题校准**：先抛痛点/反差，不含项目内部词；遵守用户对命名的任何特别要求（如某些词不许出现在标题里）。

## Failure Handling

- If MMX fails, keep the script and composition ready, then report the exact blocked asset.
- If HyperFrames render fails, inspect/lint and fix the composition before handing back.
- If Bilibili blocks an upload or requires CAPTCHA/manual file selection, pause and tell the user the exact next click or file path.
- If a public repo URL 404s because the repo is not open yet, still include it as "即将开源" only if the user said it will be open-sourced.

---
> Source: [zhoushoujianwork/hyperframes-video-skill](https://github.com/zhoushoujianwork/hyperframes-video-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
