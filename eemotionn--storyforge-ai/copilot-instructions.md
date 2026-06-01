## storyforge-ai

> 这是一个中文网文母版仓库，不是某一本小说的正文目录。

# Final Novel 母版仓库入口

## 仓库定义

这是一个中文网文母版仓库，不是某一本小说的正文目录。

它的职责是：

1. 提供可复用的框架层。
2. 为每一本小说提供独立项目目录。
3. 保证共享方法与单书 canon 隔离。

## 首次进入顺序

1. 读 `README.md`
2. 读 `framework/AGENTS.md`
3. 判断当前任务属于：
   - 母版维护
   - 新建一本书
   - 进入某一本已有小说项目
4. 如果任务属于单书创作，进入 `projects/<book-slug>/AGENTS.md`

## 根级红线

1. 根目录不承载具体小说设定、角色、卷纲、章节正文或评测结果。
2. 不在根目录直接生成章节卡、正文或单书 canon。
3. 单书事实只能写入对应 `projects/<book-slug>/`。
4. 共享层只能定义方法、模板、流程和题材 profile。

## 路由规则

1. 母版维护：读取 `framework/` 相关文档。
2. 新建项目：读取 `framework/docs/project-bootstrap-spec.md`。
3. 单书创作：进入对应 `projects/<book-slug>/AGENTS.md`。

---
> Source: [eemotionn/StoryForge-AI](https://github.com/eemotionn/StoryForge-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
