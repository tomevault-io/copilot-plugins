## nexus

> - 本文件适用于整个仓库；子目录可增加更具体的 `AGENTS.md`，但不得放宽这里的边界。

# NEXUS Repository Rules

## Scope and truth sources

- 本文件适用于整个仓库；子目录可增加更具体的 `AGENTS.md`，但不得放宽这里的边界。
- 规划事实来源按优先级为：`docs/PROJECT_INDEX.md` → `docs/无人机高精度目标定位项目规划 (2).md` → `docs/module-map (1).html` → 实验记录与代码。
- 规划中的“目标值”不是实测结果。只有 `experiments/runs/` 中有数据、配置、代码版本和结论的运行，才可进入答辩材料。

## Change boundaries

- 只修改当前任务涉及的目录；不要顺手重构、升级依赖或删除已有资料。
- 不覆盖 `docs/` 中已有规划和图片。若规划发生变化，新增决策记录，不直接改历史结论。
- 禁止提交真实密钥、设备序列号、个人隐私和未经授权的原始视频。
- 不把大体积原始数据、rosbag、编译产物或虚拟环境提交到 Git；使用 `data/` 和 `outputs/` 中的说明文件登记外部位置与校验信息。

## Mainline over defensive engineering

- 本项目目标单一：尽快验证并展示无人机目标的高精度定位。代码应优先打通 UWB、视觉、坐标转换、融合和演示主链路。
- 不要为了“完整性”堆叠大量防御性分支、抽象层、配置项或边界检查；对明确受控的实验输入，采用直接、可读、可替换的实现。
- 鼓励在实验隔离、可回滚、结果可记录的前提下采用大胆方案，先做出可测量的原型，再根据真实失败证据补检查。
- 仍必须保留最小必要保护：安全相关条件、单位/坐标系不一致、数据覆盖、凭据泄露和会让实验结果无法解释的错误不能被静默忽略。
- 发现“可以更工程化”但不影响当前演示主线的问题，只记录为后续事项，不阻塞主线实现。

## Naming

- 代码、ROS 包、配置和脚本使用 ASCII `lower_snake_case`。
- 文档和记录使用 `YYYY-MM-DD_<type>_<slug>.md`；实验运行使用 `YYYY-MM-DD_E<三位编号>_<slug>`。
- 图表使用 `fig_<topic>_<condition>_v<两位版本>.<ext>`，表格使用 `tbl_<topic>_v<两位版本>.csv`。
- 不确定的物理量、坐标系、时间戳和单位必须在文件或元数据中明确。

## Validation

- 提交前至少运行与改动对应的检查，并在记录中写明命令、环境、结果和未验证项。
- ROS1 代码变更至少通过格式检查、catkin 构建和相关单元/集成测试；ROS2 代码变更至少通过 ament/colcon 构建和相关测试；bridge 变更另外验证消息映射与时间/坐标单位。实验变更至少能由记录重跑或说明阻塞原因。
- 不能把“代码已写”表述为“精度已达标”。

## Git hygiene

- 保留其他成员的未提交修改；提交前检查 `git diff` 和 `git status --short`。
- 生成物只在确认需要交付时复制到 `defense/`，并保留来源运行编号。

---
> Source: [wblxr408/NEXUS](https://github.com/wblxr408/NEXUS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
