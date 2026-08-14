## hyperframes-music-playlist-template

> 这是一个从 workspace 模板复制出来的独立 HyperFrames 视频项目。

# HyperFrames 合成项目

这是一个从 workspace 模板复制出来的独立 HyperFrames 视频项目。

## 命令

```bash
npm run dev
npm run check
npm run render
npm run publish
```

`npm run dev` 是长时间运行的预览命令。编辑期间保持它在后台运行。

## 项目结构

- `index.html` — 主合成文件，负责页面结构和本地资源引用
- `assets/styles/main.css` — 主样式文件，负责页面布局、视觉层次和动效样式
- `assets/scripts/main.js` — 主脚本文件，负责播放逻辑、歌词同步、时间轴更新和音频可视化数据映射
- `compositions/` — 子合成
- `assets/` — 可直接用于渲染的媒体资源
- `meta.json` — 项目元数据

## 规则

1. 每个有时间轴的元素都必须包含 `data-start`、`data-duration` 和 `data-track-index`。
2. 可见的时间轴元素必须带有 `class="clip"`。
3. GSAP 时间线必须保持 paused 状态，并注册到 `window.__timelines`。
4. 只使用本地资源，避免 CDN 和网络 fetch。
5. 用户提供的图片处理成可渲染文件后，删除 workspace 中 `assets/avatars/` 目录下的原始图片，只保留处理后的输出，例如 `assets/avatars/fixed/` 下的文件。
6. 如果 LRC 是双语歌词，按 LRC 原始时间轴逐条处理。歌词显示必须基于当前激活元素的实际位置计算，不要依赖固定行高。
7. 如果歌词过长导致自动换行，不能继续按单行固定高度显示。需要给换行歌词分配足够的高度、合适的 `line-height` 和垂直间距，确保两行文字完整可见、不被裁切；歌词滚动也必须基于当前激活元素的实际位置计算，不能依赖固定行高或固定步长。修改后必须通过 `npm run check`，重点确认 inspect 没有 `clipped_text` 或布局溢出。
8. 编辑 `.html` 合成文件后必须运行 `npm run check`。
9. 除非用户明确要求渲染，否则不要运行渲染命令；迭代编辑时可以运行 preview、check、build。
10. 生成的视频是面向网友发布的成片，所有画面文案都必须是观众向表达。不要出现制作过程、素材替换、版本选择、文件来源、"这次素材"、"之前使用"、"本次使用" 等内部说明性文字；需要改写成自然的歌曲介绍、情绪描述或背景信息。
11. `playlist.json` 中 `background.lines` 是给观众看的当前歌曲介绍，只能描述当前歌曲本身；不要写歌单编排、剪辑位置或制作意图，例如“放在最后”“放在结尾刚刚好”“放在歌单中间”“这份歌单”“整份歌单”等。三行文案的第一行必须查找并使用当前歌曲准确的背景信息，简要介绍歌曲背景，例如原唱/发行信息/影视或专辑关联/创作背景等；不能凭感觉编造背景。第二、三行再写歌曲本身的情绪、画面、主题、听感或记忆点。第一行不要和 `background.meta` 重复罗列同一组信息，而要写成完整的歌曲背景故事句。
12. 每个视频 workspace 都应作为独立 HyperFrames 项目使用，包含自己的 `index.html`、`data/playlist.generated.json`、素材和渲染输出。根目录 `index.html` 只作为新 workspace 的模板源；`build/check/render` 应实际操作 `workspace/<video_id>/index.html`，避免不同视频互相覆盖。
13. 仓库只保留 `workspace/video_demo` 作为可提交模板；`workspace/video_001`、`workspace/video_002` 等实际视频工作区都是本地生成项目，不应提交到 git。不要主动把非 demo workspace 加入暂存区。
14. 导入新视频素材时，统一在目标 workspace 内按顺序落成 `assets/audio/song-XX.mp3`、`assets/lyrics/song-XX.lrc`、`assets/avatars/fixed/song-XX.jpg`，`playlist.json` 只引用这些 workspace 相对路径。源文件名可以是中文、英文、大小写扩展名或不同图片格式，但进入 workspace 后保持统一命名。
15. 封面图片不论源格式是 `jpg`、`png`、`webp` 或其他可读格式，都处理为可渲染的正方形 `800x800` JPG，输出到 `assets/avatars/fixed/song-XX.jpg`。不要在 workspace 的 `assets/avatars/` 根目录保留用户提供的原始图片。
16. 同步素材后必须校验资源与歌曲顺序：音频、歌词和封面数量要与 `playlist.json` 歌曲数量一致；每首歌的 `audio`、`lyrics`、`cover` 必须指向对应歌曲的文件；能做哈希或等效校验时，确认音频和 LRC 与用户提供的源文件一致，封面输出存在且可读。
17. 如果用户没有提供某首歌的 LRC，允许为该曲创建最小空 LRC 文件以保证构建流程可用，并在最终说明中明确告知该曲无歌词源文件。不要凭空编写歌词。
18. 用户指定歌曲顺序时必须严格按用户给出的顺序排列；重排时移动完整歌曲对象和对应资源引用，不能只改标题或只改显示文本。
19. 歌曲显示标题可以与源文件名不同，例如用户要求显示简称、中文名或常见译名时，以用户指定标题为准；但资源映射必须准确对应原始歌曲文件，并在需要时保留正确歌手信息。
20. `background.meta` 用于简短、准确的歌曲身份标签，优先写成“年份/专辑/影视/曲风”等短标签，例如“2003 · 电影情歌”“2015 · 叙事情歌”。它像目录标签，只负责让观众快速识别歌曲身份；不要和 `background.lines[0]` 重复成完整说明句，不写制作说明、素材来源或歌单位置。
21. 歌词解析只过滤包含 `和声|录音|混音|制作人` 的制作信息行；不要过滤“作词”“作曲”“编曲”等歌词署名信息，除非用户另行要求。
22. 所有会出现在画面里的文字都不能溢出或被裁切，包括左侧歌名、当前歌曲标题、底部章节标签、信息面板文案和歌词。遇到长标题或长词句时，需要用换行、缩小字号、截断省略或布局调整解决，并通过 `npm run check` 验证没有 `clipped_text` 或布局溢出。
23. 当歌曲数量超过 10 首时，左侧列表优先隐藏歌曲作者或使用更紧凑样式，确保所有歌曲条目都能显示完整，不要让最后几首被挤出画面。
24. 长视频优先使用分段检查和分段渲染：`npm run check -- --workspace <video_id>` 应能按歌曲边界分段 inspect；渲染时优先建议 `npm run render:parts -- --workspace <video_id>`，避免长时间全片 render 失败。
25. 当前模板包含单曲音频可视化：`scripts/build-playlist.js` 在 build 阶段用 `ffmpeg` 预分析每首歌音频能量和当前帧频谱，写入 `playlist.generated.json` 的 `visualizer.peaks` 与 `visualizer.spectrumFrames`；页面在当前歌曲进度条上方显示与进度条等宽的动态柱状图。柱状图表达当前播放时刻的一组频段状态，不表达整首歌的波形缩略图。`spectrumFrames` 当前使用 `16000Hz` 采样率、`1024` 点 FFT 和人声音乐优化的 54 频段布局：`60-160Hz` 8 根、`160-500Hz` 10 根、`500-2500Hz` 22 根、`2500-6000Hz` 10 根、`6000-8000Hz` 4 根。频谱值应保留固定尺度下的 FFT 频段幅度，不做每个频段自己的归一化、gamma 曲线或前后帧插值；如需放大视觉幅度，优先调整 `assets/scripts/main.js` 的统一线性显示增益，除非用户明确要求重新加入视觉增强。柱状图背后的低频光带只能作为辅助层，使用当前帧低频 band 的线性平均值驱动，不替代柱状频谱主体。不要改成浏览器实时 Web Audio 分析，以免预览、inspect、分段渲染和最终渲染不一致。调整可视化时必须保留构建期预分析数据链路，并通过 `npm run check -- --workspace <video_id>` 验证。
26. 音频可视化支持 `visualTheme` 情绪主题字段，可选 `melancholy`、`warm`、`nostalgic`、`romantic`、`calm`。如果 `playlist.json` 未配置，构建脚本会根据歌名、歌手和 `background` 文案关键词自动推断主题。调整主题配色时优先修改 `assets/scripts/main.js` 的 `VISUAL_THEMES`，调整进度条和柱状图外观时修改 `assets/styles/main.css`，调整推断规则时修改 `scripts/build-playlist.js`，并同步更新 README。
27. 切歌转场、章节光标和单曲进度播放头必须由当前播放头时间驱动，优先维护 `assets/scripts/main.js` 中的 `updateSceneEffects()`、`updateTimelineCursor()` 和 `updateSingleCursor()`；不要使用独立实时定时器或随机动画，以免预览、分段 inspect 和最终渲染不一致。底部和单曲简约呼吸播放头的光点、短脉冲和边界放大也必须由当前播放时间或预分析音频能量推导。新增可见转场元素时必须带 `class="clip"`、`data-start`、`data-duration`、`data-track-index`，并在 `scripts/build-playlist.js` 的 duration 更新列表中同步维护。当前模板不使用独立片头或片尾收束，避免开场、结尾信息变化过重。
28. 歌曲背景信息面板是观众向的唱片内页式信息，不写制作说明；样式可使用当前 `visualTheme` 变量做强调线、分隔线或标签，但必须保证 `info-title`、`info-meta` 和三行 `info-line` 不溢出、不裁切，并通过 `npm run check -- --workspace <video_id>` 验证。
29. 修改或新增代码时，应补充必要但克制的中文代码注释，说明非显而易见的数据处理、时间轴同步、布局约束或渲染稳定性原因；同时更新 README、AGENTS 或相关文档，记录新增功能、命令、约束和验证方式。不要添加空泛注释，也不要让文档与实际模板行为脱节。
30. 每次新增、修改或移除功能时，必须同步更新 `docs/feature-changelog.md`，记录需求来源、变更内容、影响范围、验证方式和回溯备注。视觉方案、交互方案、构建数据链路、模板规则、workspace 制作流程发生变化时也要记录；纯格式化或无行为变化的整理可以不记。
31. 后续回答默认在最终回复中保留可回溯的过程摘要，包括需求理解、处理路径、关键依据、修改文件、验证结果、风险或后续注意事项；回答应简洁，不输出内部思维链或未经整理的推理草稿。即使在新对话窗口中，只要仍在本项目上下文内，也应遵守该规则。
32. 模板样式已从 `index.html` 抽离到 `assets/styles/main.css`，主播放逻辑已抽离到 `assets/scripts/main.js`；调整布局、颜色、字号、进度条、歌词、唱片内页信息面板或音频可视化外观时优先修改 CSS 文件。调整播放头时间、歌曲切换、歌词同步、音频可视化数据映射或主题变量时修改主脚本文件，并保持根模板与 `workspace/video_demo` 的结构一致。`index.html` 中保留的少量内联脚本只用于兼容 HyperFrames 0.6 lint 的时间轴注册扫描，不承载实际播放逻辑。

---
> Source: [yuqianglianshou/hyperframes-music-playlist-template](https://github.com/yuqianglianshou/hyperframes-music-playlist-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
