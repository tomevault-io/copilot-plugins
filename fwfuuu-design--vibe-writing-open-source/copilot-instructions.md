## vibe-writing-open-source

> 每次开启浏览器验证当前阶段的时候，完成验证需要关闭浏览器。

## 用户指示

每次开启浏览器验证当前阶段的时候，完成验证需要关闭浏览器。

开发与部署过程中新增或生成的所有测试文件均只保留在本机，不得提交或推送到远端仓库。范围包括测试代码、测试数据库、测试账号与密钥、测试夹具、截图、日志、报告、浏览器配置及临时构建包。任务产物统一放到每次新建的 `tasks/` 子目录，并由 Git 忽略。

不得使用 `git add -f` 绕过忽略规则。每次提交或推送前必须检查暂存区及待推送提交的文件清单，排除上述文件；已有跟踪的测试文件也不能仅依赖 `.gitignore`，其后续修改同样不得随工作提交上传。不得为此擅自改写历史或删除远端已有文件。

2026-09-12 用户已明确授权从远端最新版本删除已有测试文件，保留历史记录与本机副本。删除测试文件的提交属于本次授权；应用或开发入口实际依赖的共享代码不可仅因位于 `fixtures/` 就删除。

---
> Source: [fwfuuu-design/vibe-writing-open-source](https://github.com/fwfuuu-design/vibe-writing-open-source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
