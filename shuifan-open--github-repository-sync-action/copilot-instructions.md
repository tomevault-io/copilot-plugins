## github-repository-sync-action

> - 你是一个资深的 github action 插件开发者，擅长使用 TypeScript 编写 github action 插件。

# 前置条件
- 你是一个资深的 github action 插件开发者，擅长使用 TypeScript 编写 github action 插件。
- github action 文档：https://docs.github.com/en/actions/

# 项目介绍
- 本项目是一个使用 TypeScript 创建 GitHub Action 的模板。
- 该模板提供了编译、测试、验证工作流、发布和版本控制的支持。

# 技术架构
- 使用 TypeScript 编写核心代码。
- 使用 GitHub Actions 工具包进行 CI/CD 操作。
- 使用 ncc 打包 TypeScript 代码以供分发。
- node 版本：20

# 目录结构
- src/: 包含核心 TypeScript 源代码。
- __tests__/: 包含测试代码。
- .github/workflows/: 包含 GitHub Actions 工作流配置。
- script/: 包含辅助脚本，如发布脚本。

# 项目目标
- 提供一个 github action 插件，完成代码仓库的自动同步
- 插件需要支持多种代码仓库类型，如 github、gitlab、gitee 等。
- 插件需要支持git链接，如https://github.com/xxx/xxx.git
- 从variables中获取source_reopsitory_url_list, target_repository_url_list
- 从secets中获取source_repository_username, source_repository_password
- 从secets中获取target_repository_username, target_repository_password
- source_repository_username, source_repository_password, target_repository_username, target_repository_password 可以为空，如果为空，则不用输入
- source_reopsitory_url_list, target_repository_url_list 为一对一关系，source_reopsitory_url_list 中的每个仓库需要与 target_repository_url_list 中的每个仓库进行同步
- source_reopsitory_url_list, target_repository_url_list 为多行文本，每行为一个仓库的url
- 每一对仓库克隆的以source_repository_url_list 中的仓库名为文件夹名，克隆到本地后，将克隆的仓库推送到 target_repository_url_list 中的仓库
- 每一对仓库同步不影响其他仓库的同步
- 对每一对仓库进行并发同步
- 需要保存运行日志，方便排查问题

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuifan-open) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
