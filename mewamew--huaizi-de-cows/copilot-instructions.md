## huaizi-de-cows

> 用Godot 4.x + GDScript 开发2D 小游戏《花子的奶牛关》。

# AGENTS.md

## 目标

用Godot 4.x + GDScript 开发2D 小游戏《花子的奶牛关》。

## 仓库结构

- `spec/`：当前版本目标、产品范围和交付规格。
- `assets/`：游戏资源文件。说明见`assets/README.md`
- `docs/`: 记录和项目有关的文档。如果有新增的文档，必须更新 `docs/index.md`
- `docs/experience-library/`：开发经验，已解决问题、复发规则和开发经验。
- `scenes/`：Godot 场景文件。
- `scripts/`：GDScript 游戏逻辑、调试入口和测试脚本。

## 实现约束

- 不替换 Godot 作为游戏主体。
- 外部脚本只用于资源处理、检查和辅助验证，不承载核心游戏逻辑
- 实现视觉可读性时，优先使用正式美术、动画、粒子、短暂反馈和运行时 UI 资产；不要用常驻几何圈、矩形框、文字标签或辅助线替代美术表达。
- 如果为了测试需要显示碰撞、攻击范围、激活范围、敌人类型、地图边界、刷怪区、坐标或对象池信息，必须做成默认关闭的调试辅助层，并通过显式调试参数或验证入口开启。
- 正常运行、正式视觉验收截图和交付说明默认不显示调试辅助层；带调试辅助层的截图必须在文件名或报告里明确标注。

## 子 Agent 协作

- 子 Agent 使用规则见 `docs/subagent-guide.md`；写代码的 AI 和挑毛病的 AI 分离规则见 `docs/builder-reviewer-separation.md`。
- 本项目默认使用 `.codex/agents/` 下的固定角色：
  - `builder`：根据主 Agent 指定范围写游戏实现，可改代码、场景和必要资源，不改验收标准，不做复核。
  - `test-author`：根据验收标准写或维护测试，不写实现代码。
  - `acceptance-checker`：只核对每条验收标准是否都有真实测试覆盖，不写代码、不写测试。
  - `visual-reviewer`：对照画面基准和真实运行截图做只读视觉验收，不写代码、不修 UI。
  - `reviewer`：开发完成后只读复核，先核对验收标准，再看代码质量。
- 同一轮任务里，写实现的 Agent 不能担任 `visual-reviewer` 或 `reviewer`；写实现的 Agent 也不能为了让测试通过而降低验收标准或测试要求。
- 涉及 HUD、升级弹窗、结算面板、VFX、场景观感或画面基准的任务，测试通过后必须先交给 `visual-reviewer` 只读复核；视觉复核通过后再交给 `reviewer` 做综合复核。
- 派发子 Agent 前，主 Agent 必须给出 spec、验收标准、测试入口、允许读写范围和返回要求。
- 最终结论由主 Agent 负责，但不能只照搬子 Agent 的结论；必须统一复核改动并运行必要验证。

## 必须严格遵守的开发流程

- step 1: 主 Agent 先读取 `docs/development/acceptance-standard.md`，并把它作为本项目当前验收标准；验收标准是外部合格线，AI 不得自行新增、改写或降低。如果发现标准缺失、冲突或有待定项，必须先停下来请用户确认。
- step 2: 由 `test-author` 根据 `docs/development/acceptance-standard.md` 设计详细测试用例，同样输出到 `docs/` 目录。
- step 3: 由 `acceptance-checker` 核对每条验收标准是否都有真实测试覆盖；如果没有覆盖完整，回到 step 2。
- step 4: 设计和开发测试所必须依赖的工具，建立充足的观测手段，包括但不限于日志、测试接口、截图、帧序列或可复现运行步骤。
- step 5: 根据 spec 开发游戏实现；可由 `builder` 承担实现任务。实现 Agent 不修改验收标准和测试口径，除非用户明确要求。
- step 6: 运行测试和检查。如果测试不通过，汇报失败，把失败经验更新到 `docs/experience-library/`，修复后重新回到 step 6。
- step 7: 如果改动涉及 HUD、升级弹窗、结算面板、VFX、场景观感或画面基准，测试通过后必须交给独立 `visual-reviewer` 只读复核；`visual-reviewer` 与写实现的 Agent 不是同一个上下文。
- step 8: 如果 `visual-reviewer` 不通过，汇报失败并修复，必要时更新 `docs/experience-library/`，然后重新回到 step 6。
- step 9: 测试通过且必要的视觉复核通过后，必须交给独立 `reviewer` 复核；`reviewer` 与写实现的 Agent 不是同一个上下文，并且默认只读。
- step 10: 如果 `reviewer` 不通过，汇报失败并修复，必要时更新 `docs/experience-library/`，然后重新回到 step 6。
- step 11: 只有测试通过、必要的 `visual-reviewer` 通过、`reviewer` 通过、主 Agent 复核通过，才视为开发完成。

---
> Source: [mewamew/huaizi-de-cows](https://github.com/mewamew/huaizi-de-cows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
