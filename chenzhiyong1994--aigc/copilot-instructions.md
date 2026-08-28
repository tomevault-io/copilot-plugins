## aigc

> 本仓库是可复用的 AIGC 创作能力库，不是长期作品档案库。

# AIGC 项目规则

## 项目定位与事实来源

本仓库是可复用的 AIGC 创作能力库，不是长期作品档案库。

- `skills/` 定义各类任务的触发条件、执行流程和交付边界。
- `references/` 存放可复用的风格、镜头、音乐和提示词参考。
- `docs/standards/` 规定 Skill 结构与任务产物归档方式。
- `output/` 只用于当前任务、紧凑示例或 eval 支撑材料；旧目录默认都是历史记录。
- `docs/plans/` 是历史设计笔记，只有用户明确点名继续时才读取。

## 任务路由

执行前按实际任务范围选择入口，并完整阅读对应 `SKILL.md` 及其要求的 reference：

- 图片设计、提示词或实际生图：`skills/image-design/SKILL.md`；Codex 中的真实位图生成交给运行时 `imagegen`，其他环境使用等价适配器。
- 视频分镜、关键帧/首尾帧、图生视频或文生视频提示词：`skills/aigc-video/SKILL.md`。
- 收到图片、图片提示词、视频提示词或外部成片的真人评价与修改建议：`skills/aigc-feedback/SKILL.md`。
- 歌曲、歌词、曲风、单曲、EP 或专辑设计：`skills/music/SKILL.md`。
- 安装或配置网易云 CLI：`skills/ncm-cli-setup/SKILL.md`。
- 网易云搜索、播放和队列控制：`skills/netease-music-cli/SKILL.md`。
- 基于偏好的网易云推荐与助手行为：`skills/netease-music-assistant/SKILL.md`。
- 新建或维护项目 Skill：先读 `docs/standards/skill-structure.md`。
- 保存任务文件或判断归档边界：先读 `docs/standards/output-archive.md`。

不要为单一任务额外引入总控 Skill，也不要默认升级为重型交付包。

## 必须遵守的跨域护栏

- 本项目不维护图片模型路由、Token Probe、备用通道或渠道选择表；没有可用位图生成适配器时只交付提示词，不得伪造生成结果。
- 只有真实文件存在时才能记录路径或声称完成生成；遇到阻塞应记录真实原因，不得伪造图片、视频、音乐或日志。
- 视频与音乐默认交付设计、提示词或素材包。除非确有可用适配器并实际执行，否则不得声称已经生成成品。
- `output/`、`.aigc-local/`、`.venv-audio/` 和历史 plan 不得发布，也不得作为其他用户的新任务默认规则；本地学习规则只按当前范围读取。
- 不得把 `appId`、`privateKey`、Cookie、Token、登录二维码或其他凭证写入仓库。
- 受 Git 跟踪的反馈规则只保留匿名化的 `confirmed` 公共基线；`candidate`、个人偏好、客户语境、原始路径和日期证据写入已忽略的 `.aigc-local/`。
- 项目文档和素材包正文优先使用中文；命令、标识符、平台参数和必要的生成提示词可保留英文。

## 维护与验证

- 可复用行为变化时，修改最近的 Skill、reference、eval 或标准文档；不要把详细流程复制回本文件。
- 长示例和平台细节放入 reference，`SKILL.md` 只保留触发条件、核心流程、读取路由与输出边界。
- 修改 `SKILL.md` 后执行最小 frontmatter/结构校验；保存视频提示词时按 `skills/aigc-video/SKILL.md` 运行引用校验。

---
> Source: [chenzhiyong1994/AIGC](https://github.com/chenzhiyong1994/AIGC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
