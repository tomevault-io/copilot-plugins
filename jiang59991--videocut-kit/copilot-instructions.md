## videocut-kit

> 本仓库是一个**口播视频剪辑** skill（文稿驱动、人在环；Claude Code 与 Codex 等 agent 通用）。

# AGENTS.md

本仓库是一个**口播视频剪辑** skill（文稿驱动、人在环；Claude Code 与 Codex 等 agent 通用）。
本文件只是**入口指针**——完整流程与约束在 [SKILL.md](SKILL.md) / [WORKFLOW.md](WORKFLOW.md)。

## 触发

用户说「剪口播 / 删口水词 / 校稿 / 处理这个口播视频」时：
先读 **SKILL.md §0 分支地图** 判断四轴（输入 `.screenstudio`/MP4、模式 细致/auto、画幅、阶段），
再用 **`node scripts/pipeline.js`（单一入口）** 按暂停点推进。
**不确定当前在哪一步 / 接手别人的活 / 上下文丢了：`node scripts/pipeline.js output/<name> --status`**——
它回显 📍当前位置 + 👉下一条确切命令 + doctor 体检（SS 流还会回显「下一关 SS-x」）；细节见 SKILL「🧭 上下文丢失/冷启动续跑」节。

## 你的职责（pipeline 跑确定性步骤，你做按语境的判断）

- **首次上手**（第一次给这位用户干活时）：带他做两件可跳过的一次性设置——① 转写 API（`.env` 没配就引导填
  火山 `VOLC_API_KEY` / Groq `GROQ_API_KEY`，🔴 不读 `.env` 值）；② 主题色（有品牌色就改 `brand.css` 的
  `--accent`/`--accent-dim`/`--accent-edge` 一键个性化组件，没有就用默认钴蓝）。详见 SKILL「首次上手」节。
- **⏸ A** 通读全文判删除（**先读 `rules/preferences.md`**）→ 写 `edit.decisions.json`。
- **auto 模式**（`--auto`）：剪辑由 `auto_cut.js` 自动执行后，**必须把 `auto_cut.report.md`
  的删除清单+理由在聊天里报给用户**；最后在画面预览一次性验收，🔴 用户确认后才渲染。
  用户对成片要求高时主动建议细致模式。
- **⏸ C** 字幕分句 + 加标点（🔴 不改字、不翻译英文术语）→ `--phase captions` 回填（自动画幅适配）。
  字幕**非人工审核**：拿不准的断句/用字在聊天里主动问用户，其余直接定稿。
- 纠同音字（可选）：据用户参考脚本产 `corrections.json` → `correct_tokens.js`（等长替换）。
- **⏸ E** 建合成基座、提组件建议（只用 `COMPONENTS.md` 现成 `fresh-*`），snapshot 验收。
- **⏸ F**（成片渲染验收后）：跑 `node scripts/distill_prefs.js output/<name>` 做偏好复盘 → 列七列候选表
  → **停下问用户哪些要写回** → 按 8 类分区写回 `rules/preferences.md`（confirmed/observing）。不等圈选绝不落笔。
- 每个阶段结束写 `output/<name>/learning/*.md` 复盘，按铁律沉淀偏好；用户当场说「以后都这样/固化」即记入「待沉淀」区。

## 最致命的硬约束（全集见 SKILL.md「关键约束」）

- **🔒 字幕硬门卡**：机器初稿（`source:'auto'`）会被拒绝进成片——必须过 ⏸C AI 分句。
- **删前保后**；逐字时间戳为权威、绝不吃字。
- **SS 输入流**：SS 导出关卡（用户在 Screen Studio 复核并导出成片，SS-2）之前，绝不搭合成/加组件。
  ⏸D 之后的续跑指向 `ss_apply.js` 写回 + 等用户导出（导出后 `--ss-exported` 确认解锁画面），**不是** `--phase storyboard`。
- 审阅必须用 `review_server.js`（`/`＝统一审阅中心）。
- **绝不读取 `.env` 内容**。
- 不提交视频/音频/`.env`；`output/`、`builds/` 不入 git。

---
> Source: [Jiang59991/videocut-kit](https://github.com/Jiang59991/videocut-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
