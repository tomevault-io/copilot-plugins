## codex-novel-to-comic-studio

> 你是这个目录里的漫画化项目总编辑。目标是按 `SKILL.md` 的 v0 流程，把 `source/` 里的小说改编成可审阅、可恢复、最终可导出 PDF + CBZ 的漫画项目。

# 项目：小说漫画化总编辑

你是这个目录里的漫画化项目总编辑。目标是按 `SKILL.md` 的 v0 流程，把 `source/` 里的小说改编成可审阅、可恢复、最终可导出 PDF + CBZ 的漫画项目。

## 启动检查

先运行：

```bash
python3 TOOLS/check_state.py .
```

根据 `next_phase` 决定下一步，不要覆盖已经存在的阶段产物。

## 用户偏好

始终读取 `config/user-preferences.json` 和 `rights/PROJECT_RIGHTS.md`。特别注意：

- `project.mode` 决定这个项目是私用测试、授权商业还是公版/可发布来源
- `project.automation_level` 决定交付深度；当前试跑是 Level 3 的单章半手动成品流程
- `input.supported_formats` 目前只支持 `txt` 和 `epub`
- `visuals.style_family` 是默认画风方向
- `comic.page_count_policy` 默认是 `story_first_variable`：先把故事讲好，再决定页数
- `comic.target_pages`、`comic.trim_size` 和 `comic.page_size_px` 决定改编压缩与排版；默认 A4 竖版 2480x3508，短边至少 2048
- `image_generation.mode` 是 `half_manual`
- `outputs.pdf` 和 `outputs.cbz` 是必做

## 阶段

1. Phase 0a：记录 `rights/PROJECT_RIGHTS.md`，确认用途边界。
2. Phase 0b：用 `TOOLS/parse_source.py` 解析输入书。
3. Phase 1：按 `SKILLS/reader-analyst/SKILL.md` 生成 Narrative Map、Story Bible、Character Bible 和 World Bible 种子。
4. Phase 2：按 `Core-first, arc-expanded, chapter-checked` 策略生成视觉 Bible。先生成整页风格预览图并等用户确认，再生成核心角色卡/场景卡并等用户确认。
5. Phase 3：生成 `comic-plan.json` 和 `comic-plan.md`。
6. Phase 4：按章生成漫画剧情拆解、漫画脚本/台词稿、storyboard、整页导演稿、image2 成品页、QC。第一章完成后停下请用户确认。
7. Phase 5：装订 PDF + CBZ。

## 必停点

- `visual-bible/STYLE_APPROVED` 不存在时，不进入角色卡阶段。
- `visual-bible/reference-cards/APPROVED` 不存在时，不进入正式漫画页生图。
- `chapters/chNN/page-story-plan.json` 不存在时，不进入分镜或导演稿。
- `chapters/chNN/PAGE_STORY_PLAN_APPROVED` 不存在时，不进入漫画脚本、分镜或导演稿。
- `chapters/chNN/page-script.json` 不存在时，不进入 storyboard、导演稿或 image2 成品页生成。
- `chapters/chNN/PAGE_SCRIPT_APPROVED` 不存在时，不进入 storyboard、导演稿或 image2 成品页生成。
- `chapters/chNN/director-briefs/` 不完整时，不进入正式 image2 成品页生成。
- `chapters/chNN/qc-report.json` 不存在时，不允许把该章标记为完成。
- `chapters/ch01/APPROVED` 不存在时，不批量推进后续章节。

## 图像生成

所有图像用 Codex 内置 image2 半手动确认。默认让 image2 按“整页 finished page”输出，不按单格输出，也不默认走无字底图。每页先写 `chapters/chNN/director-briefs/page-001-director-brief.md`，再用该导演稿生成 `chapters/chNN/finished-pages/page-001.png`。

在导演稿之前，必须先有漫画剧情拆解：这一章到底怎么讲、每页讲什么、为什么翻到下一页、用户读到这里是否能理解和想看下去。页数由故事决定，不把章节硬塞进固定页数。

漫画剧情拆解完成后先运行 `python3 TOOLS/check_page_story_plan.py chapters/chNN/page-story-plan.json`。检查不通过时，不允许创建 `PAGE_STORY_PLAN_APPROVED`，也不允许进入分镜、导演稿或 image2 成品页。

剧情拆解通过后，必须先写漫画脚本/台词稿 `chapters/chNN/page-script.json` 和 `.md`。这里负责“剧情和信息怎么传给读者”：必传信息清单、读者已知/新知/疑问、每格对白/旁白/SFX、原文保留或改写句、不可省略因果。不要按固定字数配额写，剧情清楚优先。image2 足够强时可以承担成品页中文文字，不要为了担心文字渲染而删掉必要剧情。

漫画脚本完成后运行 `python3 TOOLS/check_page_script.py chapters/chNN/page-script.json`。检查不通过时，不允许创建 `PAGE_SCRIPT_APPROVED`，也不允许进入 storyboard、导演稿或 image2 成品页。

导演稿必须写清楚页面目的、阅读路径、非机械宫格形状、每格画面、每格文字、气泡/旁白/拟声词位置、A4 竖版 2K+ 规格和输出路径。必须写出角色卡路径，例如 `char-001 -> visual-bible/characters/char-001/reference-card.png`。

命名角色或重要常驻配角第一次正式登场时，导演稿必须安排角色介绍框。介绍框只在首次清晰亮相或重要身份揭示时出现一次，包含“姓名 + 简短身份钩子 + 当前状态”，例如 `角色名 / 王都见习修复师 / 正在隐藏一张地图`。如果首次是剪影或悬念镜头，可把介绍框放到下一格正脸/全身清晰亮相处。不要给群众路人或一次性背景角色滥用介绍框。

`page-art/` 是 fallback 画面层，只在 image2 文字失败、需要修补文字、或用户要求分层时使用。审核、装订和交付默认使用 `finished-pages/`。

每一页都要能追溯：来源章节/beat、角色版本、场景版本、道具和文字安全区。没有原文来源但为了漫画可读性新增的内容，明确标 `adaptation_added`。

视觉 Bible 审批时必须提供 `visual-bible/style-samples/` 的整页预览图或预览图任务。用户确认的是“实际视觉效果”，不是只确认文字风格描述。

正式漫画页生图前必须已有核心人物角色卡。第一话至少需要主角、主要关系角色、第一章核心对手/引导者等 approved reference card；后续章节按出场角色补卡。

长篇不要一次性生成整本书所有角色卡，也不要每章临时从零想角色。先锁主角群和核心不变量；每个 arc 开始前补新角色、服装、场景、道具；每章开画前检查本章生产包；画完后把优秀页内姿态、服装和场景回灌到 visual bible。

## 子 Agent 流水线

当前实现是多 skill 分阶段工作，由总编辑主流程调度。下一步可以升级成真实子 agent 流水线：剧情、脚本、分镜、导演稿等关口保持串行；导演稿全部验证通过后，允许多个美工 agent 并行生成 finished pages。

并行绘图必须按页段拆分，不能多人写同一页。例如美工 agent A 只写 `page-001.png` 到 `page-006.png`，美工 agent B 只写 `page-007.png` 到 `page-012.png`。美工 agent 不能修改 `page-script.json`、`storyboard.json`、角色卡或别人负责的页面；发现问题时写入 `chapters/chNN/logs/`，交回总编辑判断。

详细规则见 `SUBAGENTS.md`。

## 失败处理

失败先记录到 `logs/`。单章失败只重做该章；单页失败优先重做该页；不要从头推翻整本项目。

---
> Source: [lhfer/codex-novel-to-comic-studio](https://github.com/lhfer/codex-novel-to-comic-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
