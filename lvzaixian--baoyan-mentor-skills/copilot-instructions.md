## baoyan-mentor-skills

> 本仓库用于制作可公开分发的 Codex Skills。默认使用中文编写用户说明；文件名、命令、YAML 字段和 Skill 标识可使用英文。

# Baoyan Mentor Skills 工作协议

本仓库用于制作可公开分发的 Codex Skills。默认使用中文编写用户说明；文件名、命令、YAML 字段和 Skill 标识可使用英文。

## 公共仓库边界

- 只允许原创通用规则、确定性脚本、空白模板和完全虚构的测试资料。
- 禁止复制任何私有工作区的 Git 历史、真实个人资料、真实导师记录、邮件、回复、附件、匿名评价、付费课程内容或来源未闭合的模板。
- 迁移既有逻辑前先登记来源、作者、许可和 clean-room 处理方式；来源未闭合时不得复制。
- 合成示例必须使用虚构姓名、学校、邮箱、项目、成绩和附件。
- 仓库内的 Skill 目录是待发布源代码；只有复制或安装到用户的规范 Skills 目录后才视为可用 Skill。

## 真源与状态边界

- `targets/index.md`：院校或项目申请状态真源。
- `targets/<target>/mentor-outreach/screening.yaml`：导师候选、来源、claim、评分、筛选状态和联系准备度的机器真源。
- `导师筛选总览.md`、`筛选证据/来源记录.md`、`待核实问题.md`：由 `screening.yaml` 派生的阅读视图，不得成为第二份可独立修改的状态库。
- 单导师材料状态：实际文件和 `.meta/source-notes.md`。
- 发送、回复、考核和下一步：用户确认或可核验的外部证据，加 `targets/导师套磁状态表.md`。
- 材料完成、目录存在或 `READY_FOR_USER_REVIEW` 永远不能证明已发送。

导师筛选状态不得混入 `已联系`、`待跟进` 等通信状态。

## 永久 no-send

任何 Skill、脚本、示例或文档均不得实现或引导：

- SMTP、IMAP 或邮箱账号登录；
- 自动发送、回复或跟进；
- 申请系统上传、提交或签署；
- 从材料存在性推断真实发送或回复。

所有对外动作必须停在用户审阅与自行执行之前。

## 实现要求

- 按 `docs/superpowers/plans/` 的阶段顺序实施。
- 使用 Skill Creator 的 `init_skill.py` 创建正式 Skill；在正式内容写完前不要把占位 Skill 标记为可安装。
- `SKILL.md` 保持精简，详细 schema、写作协议和示例放在一层 `references/` 中。
- 只有重复、确定性且容易出错的操作进入 `scripts/`；优先 Python 标准库，筛选 YAML 解析仅使用 PyYAML。
- 不建设数据库、GUI、邮箱集成、运行时 outreach schema、语义评分器、首封材料 manifest 或 receipt 系统。导师盲审包可保留仅含哈希、随机化和泄漏检查结果的审计 manifest，但不得含身份内容或承担运行时状态。
- 新增非平凡逻辑必须留下最小可运行测试；发布前运行 Skill 结构校验、单元测试、合成端到端测试和公开边界扫描。

## 安全写入

- 初始化脚本只写入新目录或空目录，不删除既有内容。
- 已存在且来源不明的导师简历目录标记为 `EXISTING_UNMANAGED`，不得覆盖、合并或清理。
- 盲审身份映射使用权限 `0600` 的临时文件并原子替换；输入验证成功前不得清理输出。
- 不提交密钥、身份映射、绝对用户路径、构建缓存或真实附件。

---
> Source: [lvzaixian/baoyan-mentor-skills](https://github.com/lvzaixian/baoyan-mentor-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
