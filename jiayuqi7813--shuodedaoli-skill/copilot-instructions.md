## shuodedaoli-skill

> 合成竖屏对话短视频（游戏背景 + 角色头像 + 彩色字幕 + Fish Audio TTS）。支持缺少角色图时调用 [$imagegen](/Users/snowywar/.codex/skills/imagegen/SKILL.md) 生成头像，并要求真实透明背景或纯净实底，禁止棋盘格假透明。含 YAML 脚本、音色搜索、无 TTS 预览与完整成片。Use when the user mentions dialogue-shorts、对话短视频、语录视频、熊大语录、Fish Audio TTS、或本工具包路径。


# dialogue-shorts（对话短视频工具包）

本目录即完整工具包根目录，结构：

```
dialogue-shorts/
├── SKILL.md / reference.md
├── video_framework/     # 合成核心
├── scripts/             # CLI
├── examples/            # 示例 YAML
├── assets/              # 背景、头像
└── output/              # 成片输出
```

## 依赖

- Python 3.10+、`ffmpeg`、`yt-dlp`
- `pip install -r requirements.txt`
- `.env` 中配置 `FISH_API_KEY`（API 钱包余额，与网页积分通常不互通）

## 视频五部分

| # | 模块 | 说明 |
|---|------|------|
| 1 | 背景 | 横屏源视频中心裁 9:16，`background` 指定路径，可 `background_start` 或随机切片 |
| 2 | 头像 | 1–5 人；说话者高亮；`assets/characters/{id}.png` |
| 3 | TTS | Fish Audio，`voice_id` 见 [reference.md](reference.md) |
| 4 | 字幕 | ASS 画面居中（`subtitle_alignment: 5`），按角色 `color` |
| 5 | 文案 | 手写 YAML 或由 Agent 按主题生成 |

## 角色图生成

- 如果用户没有提供角色头像，先调用 [$imagegen](/Users/snowywar/.codex/skills/imagegen/SKILL.md) 生成，再继续视频合成。
- 生成结果默认保存到 `assets/characters/{id}.png`，与 YAML 中的 `avatar` 字段保持一致。
- 优先生成适合短视频头像位的半身角色图或头像图，主体清晰、背景干净，方便高亮和叠放。
- 如果要求透明背景，必须是真实 alpha 透明，禁止输出带棋盘格、灰白格、伪透明预览底图的“假透明”图片。
- 如果模型产出了假透明背景，不能直接拿来合成；应重新生成，或先去掉棋盘格背景后再保存到项目里。
- 如果用户给了角色设定、风格参考、服装描述或现有图片，把这些信息传给 `imagegen`；如果没给，就根据角色名、台词气质和视频主题补足一个合理的角色图提示词。
- 已有头像文件时不要直接覆盖，除非用户明确要求替换；否则生成同目录新版本后再更新 YAML 引用。

## 首次：下载背景

```bash
cd <本目录>
yt-dlp -f "bestvideo[height<=1080]+bestaudio/best[height<=1080]" \
  --merge-output-format mp4 \
  -o assets/backgrounds/parkour.%(ext)s \
  "https://www.youtube.com/watch?v=u7kdVe8q5zs"
```

## 音色搜索与试听

```bash
python3 scripts/search_voices.py 熊大
python3 scripts/search_voices.py 熊二 --preview   # 官方 sample，不扣 TTS
```

将 `voice_id` 写入 YAML `characters[].voice_id`。详见 [reference.md](reference.md)。

## 语音缓存

- 完整成片会把每句 TTS 音频缓存到 `output/audio_cache/<script_stem>/`。
- 缓存目录会同时写入 `manifest.json` 和每句音频对应的 `.json` 元数据，记录台词、speaker、`voice_id` 与时长。
- 当脚本内容、speaker 和 `voice_id` 没变时，后续重新调整头像、字幕、背景或排版，应直接复用本地缓存音频，不再重复调用 Fish TTS API。
- 只有当台词文本、角色音色或缓存缺失时，才重新请求 TTS。

## 命令

```bash
# 无语音预览
python3 scripts/generate_preview.py examples/sample_script.yaml -o output/preview.mp4

# 完整成片（TTS 时长驱动字幕）
python3 scripts/generate_full.py examples/sample_script.yaml -o output/full.mp4
```

## YAML 要点

```yaml
background: assets/backgrounds/parkour.mp4
characters:
  - id: xiongda
    name: 熊大
    color: "#E8A317"
    avatar: assets/characters/xiongda.png
    position: left
    voice_id: <Fish Audio model id>
lines:
  - speaker: xiongda
    text: 台词
```

有 TTS 时无需手写 `duration`；仅预览时可按字数估算。

## Agent 工作流

1. 先确认背景、角色、台词、是否需要成片 TTS。
2. 检查 `characters[].avatar` 指向的角色图是否存在。
3. 若缺少角色图，优先调用 [$imagegen](/Users/snowywar/.codex/skills/imagegen/SKILL.md) 为每个角色生成 `assets/characters/{id}.png`。
4. 检查生成图是否为真实透明背景，不能带棋盘格假透明。
5. 补齐或更新 YAML 后，再运行预览或完整成片命令。

## Agent 检查清单

```
- [ ] 在工具包根目录执行命令
- [ ] background 存在；头像存在，或先调用 imagegen 生成到 assets/characters/
- [ ] 角色图若声明透明背景，实际文件必须是真透明，不能有棋盘格假透明
- [ ] 每角色有 voice_id（成片）
- [ ] FISH_API_KEY 已配置且 API 余额充足
```

## 安装为 Cursor Skill（可选）

```bash
ln -sf "$(pwd)" ~/.cursor/skills/dialogue-shorts
```

之后在对话中 @dialogue-shorts 引用。

---
> Source: [jiayuqi7813/shuodedaoli-skill](https://github.com/jiayuqi7813/shuodedaoli-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
