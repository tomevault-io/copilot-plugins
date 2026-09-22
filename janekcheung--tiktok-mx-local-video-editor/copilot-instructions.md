## tiktok-mx-local-video-editor

> 本项目是零额外 API 费用的本地 TikTok 墨西哥区商品视频剪辑工具。Codex 使用它时，应通过本机 Node.js 和 FFmpeg 完成剪辑，不创建 OpenAI API Key，不调用付费 API。

# Codex 工作约定

本项目是零额外 API 费用的本地 TikTok 墨西哥区商品视频剪辑工具。Codex 使用它时，应通过本机 Node.js 和 FFmpeg 完成剪辑，不创建 OpenAI API Key，不调用付费 API。

## 用户说“开始剪辑”时

1. 在用户指定的任务目录中工作；未指定时，检查当前项目下最近修改且包含 `input/materials` 的任务目录。
2. 读取 `input/edit-request.txt`，并把用户当前聊天里的最新剪辑要求通过 `--request` 传给命令行。聊天要求优先。
3. 先运行 `node src/cli.mjs analyze <任务目录> --request "<聊天要求>"` 检查素材和内部计划。
4. 只要存在至少一个自有素材，且有对标视频或文字剪辑要求，就继续运行 `preview`。
5. 缺少音乐、口播、商品链接或字幕不得阻止初剪。缺少商品资料时，不添加推测性卖点。
6. 只有用户明确回复“确认导出”或同义表达后，才运行 `final <任务目录> --confirmed`。

## 剪辑原则

- 有文字要求时，文字优先；对标视频只作为结构、节奏、风格参考。
- 没有对标时，根据文字要求和安全默认值生成时间线。
- 默认 TikTok 9:16 竖屏，20-30 秒；若用户指定时长则按用户要求。
- 无商品信息、无口播时，不添加推测性字幕、价格、优惠、销量、功效、认证或库存。
- 音乐顺序：本次音乐、任务曲库、项目曲库、用户手动取得的 TikTok Commercial Music Library 音乐、素材原声、无声。
- 不自动抓取或下载 TikTok 音频。
- 不默认复用对标音轨；只有用户明确要求并自行确认授权风险时才复用。
- 素材反光或过曝时，可使用防反光/压高光调色，优先保证商品细节清晰。

## 输出规则

- `output/` 只交付视频文件，例如 `preview.mp4`、`final.mp4` 或多条正式视频。
- 剪辑计划、素材报告、时间线、接触图和字幕转写只作为内部工作文件，不放进 `output/`。
- 如果视频画面或口播里出现西语，不生成额外字幕或翻译文件；需要中文对照时，直接在聊天框回复。

## GitHub 使用场景

如果用户给出本项目 GitHub 地址并要求下载使用：

1. 克隆仓库到本地。
2. 运行 `npm run check` 检查 Node.js、FFmpeg、FFprobe。
3. 用 `node src/cli.mjs init tasks/<任务名>` 新建任务目录。
4. 指导用户把素材放入 `input/materials`，对标视频放入 `input/reference`，文字要求放入聊天或 `input/edit-request.txt`。
5. 按上面的剪辑流程生成初剪和正式成片。

## 禁止事项

- 不创建或要求用户创建 OpenAI API Key。
- 不调用 OpenAI API、Groq API、云端转写、云端剪辑或付费配音服务。
- 不上传用户素材到外部服务。
- 不把 `tasks/`、模型文件、真实素材或成片提交到 GitHub。

---
> Source: [JaneKCheung/tiktok-mx-local-video-editor](https://github.com/JaneKCheung/tiktok-mx-local-video-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
