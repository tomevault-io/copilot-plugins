## class2data

> 本仓库用于持续开发”课小记”。目标是交付一个手机优先、本地数据优先的家庭课外班账本和成长档案。

# AGENTS.md

本仓库用于持续开发”课小记”。目标是交付一个手机优先、本地数据优先的家庭课外班账本和成长档案。

## 语言与沟通

- 始终使用中文思考、记录和回复。
- 对事实、推测和待确认事项要分开说明。
- 不确定时先查证本仓库文档和代码；仍不确定再向用户提问。

## 必读入口

按需懒加载文档，不要一次性把所有资料读完。

- 每次对话快速定位：[docs/engineering/project-status.md](docs/engineering/project-status.md)
- 开工定位流程：[docs/engineering/agent-startup.md](docs/engineering/agent-startup.md)
- 端到端实施流程：[docs/engineering/delivery-lifecycle.md](docs/engineering/delivery-lifecycle.md)
- 产品需求源文档：[docs/interest-class-app-design.md](docs/interest-class-app-design.md)
- 原始用户故事：[docs/story.md](docs/story.md)
- 工程文档导航：[docs/engineering/README.md](docs/engineering/README.md)

每次新对话或上下文恢复后，先读 `project-status.md` 判断当前阶段、下一步和阻塞项，再按任务类型读取相关文档。

当任务涉及需求、UI、架构、数据模型、业务规则、实现计划、验收标准或发布时，优先读取 `docs/engineering/` 下对应文档。

## 技术基线

第一版技术路线固定为：

- Flutter
- Riverpod
- go_router
- Drift + SQLite
- App 私有目录
- 本地通知
- 手动备份与恢复

除非用户明确要求，不要把第一版扩展成云同步、多用户协作、Web 后台或培训机构管理系统。

## 包管理与环境

- Python 相关操作使用 `uv`，默认 Python 版本为 3.12。
- Node.js 相关操作使用 `pnpm`，禁止使用 `npm` 或 `yarn` 作为项目包管理器。
- Flutter/Dart 依赖使用 Flutter/Dart 官方命令管理，例如 `flutter pub get`、`dart run build_runner build`。
- 不要在没有项目配置的情况下随意初始化新技术栈；先检查现有文件和工程结构。

## 工作流程

每次接到非平凡任务时：

1. 读取 `project-status.md`，确认当前生命周期阶段和推荐下一步。
2. 判断任务属于需求设计、产品规格、交互/UI、技术设计、开发、测试验收还是发布。
3. 读取与该阶段直接相关的产品和工程文档。
4. 检查当前文件结构和已有实现，确认是否已有可复用模块。
5. 制定简短计划，明确本次任务边界、涉及文件、产物和验收方式。
6. 按阶段产物或垂直功能切片推进，保持改动集中。
7. 修改完成后运行对应验证命令或文档一致性检查。
8. 更新必要文档和 `project-status.md`，记录新增约束、设计决策或未完成事项。
9. 交付时说明改动、验证结果和剩余风险。

## 不可破坏的产品规则

- 日常记录路径必须尽量短，目标是不超过 10 秒。
- 一次购课或续费必须形成一个新的课包。
- 课时余额由课包和课时流水计算，不靠单个余额字段手工维护。
- 默认值只是减少填写，最终以本次上课记录为准。
- 上课计划不等于实际上课记录，计划课应运行时生成近期待处理项。
- 照片和附件默认进入 App 私有目录，不默认写入系统相册。
- 备份和恢复属于第一版核心功能。
- 系统语义类型不能被用户随意改名或新增业务含义。
- 普通分类标签允许自定义，但历史记录必须保留当时显示名称。

## 质量门禁

实现代码后至少执行与改动匹配的验证：

- `dart format .`
- `flutter analyze`
- `flutter test`
- 涉及 Drift/Riverpod 代码生成时执行 `dart run build_runner build --delete-conflicting-outputs`

如果某项命令因项目尚未初始化、环境缺失或平台限制无法运行，必须在交付时明确说明。

## 文档维护

- 新增架构或流程约束时，更新 `docs/engineering/`。
- 需求变化影响第一版范围时，更新 `docs/interest-class-app-design.md` 或新增决策记录。
- 不要把临时想法混进稳定规范；未确认内容写入“待确认”或“后续演进”。

---
> Source: [ytygxfmgzx/class2data](https://github.com/ytygxfmgzx/class2data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
