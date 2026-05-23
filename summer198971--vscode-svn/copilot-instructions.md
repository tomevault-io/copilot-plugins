## vscode-svn-plugin

> 这是一个VSCode SVN插件项目，继承了SVN基本功能并增加了AI提交日志功能。项目主要文件：

# VSCode SVN 插件开发规则

## 项目概述

这是一个VSCode SVN插件项目，继承了SVN基本功能并增加了AI提交日志功能。项目主要文件：

- [package.json](mdc:package.json) - 插件配置和依赖
- [README.md](mdc:README.md) - 功能文档和使用说明
- [CHANGELOG.md](mdc:CHANGELOG.md) - 版本更新记录
- [src/extension.ts](mdc:src/extension.ts) - 插件主入口文件
- [src/svnService.ts](mdc:src/svnService.ts) - SVN操作核心服务
- [src/svnLogPanel.ts](mdc:src/svnLogPanel.ts) - SVN日志查看面板
- [src/commitPanel.ts](mdc:src/commitPanel.ts) - 文件提交面板
- [src/folderCommitPanel.ts](mdc:src/folderCommitPanel.ts) - 文件夹提交面板
- [src/aiService.ts](mdc:src/aiService.ts) - AI提交日志生成服务

## 开发规范

### 核心要求
1. **中文优先**: 所有代码注释、用户界面文本、错误信息必须使用中文
2. **右键菜单集成**: 确保所有SVN操作命令都出现在右键上下文菜单中
3. **用户体验**: 优先考虑视觉表现和用户友好的交互体验
4. **文档维护**: 所有功能添加和修改必须记录在 [README.md](mdc:README.md) 和 [CHANGELOG.md](mdc:CHANGELOG.md) 中

### 代码风格
- 使用TypeScript编写所有代码
- 遵循VSCode扩展开发最佳实践
- 函数和变量命名使用英文，但注释使用中文
- 错误处理要完善，提供用户友好的中文错误信息

### 功能模块

#### 1. SVN基础操作 ([src/svnService.ts](mdc:src/svnService.ts))
- 文件/文件夹上传提交
- 文件更新和恢复
- 工作副本路径管理
- SVN状态检查和信息获取

#### 2. 用户界面面板
- **提交面板** ([src/commitPanel.ts](mdc:src/commitPanel.ts)): 单文件差异对比和提交
- **文件夹提交面板** ([src/folderCommitPanel.ts](mdc:src/folderCommitPanel.ts)): 批量文件提交管理
- **日志面板** ([src/svnLogPanel.ts](mdc:src/svnLogPanel.ts)): SVN历史记录查看，支持本地版本对比
- **更新面板** ([src/updatePanel.ts](mdc:src/updatePanel.ts)): 文件更新进度显示

#### 3. AI集成 ([src/aiService.ts](mdc:src/aiService.ts))
- 支持OpenAI和通义千问模型
- 基于文件差异自动生成提交日志
- API密钥配置管理

### 配置管理

#### package.json 配置要点
- `commands`: 定义所有SVN操作命令
- `menus`: 配置右键上下文菜单和命令面板
- `configuration`: 定义用户可配置的设置项
- `activationEvents`: 定义插件激活条件

#### 重要配置项
- `vscode-svn.customSvnRoot`: 自定义SVN工作副本路径
- `vscode-svn.aiModel`: AI模型选择 (openai/qwen)
- `vscode-svn.aiApiKey`: OpenAI API密钥
- `vscode-svn.qwenApiKey`: 通义千问API密钥
- `vscode-svn.showUncommittedChanges`: 显示未提交更改

### 开发注意事项

1. **版本管理**: 每次功能更新都要更新 [package.json](mdc:package.json) 中的版本号
2. **菜单配置**: 新增命令时必须同时配置在 `explorer/context` 和 `editor/context` 菜单中
3. **错误处理**: 所有SVN命令执行都要有完善的错误处理和用户提示
4. **编码问题**: 特别注意SVN操作中的中文编码问题，设置UTF-8环境变量
5. **WebView安全**: 使用WebView时要注意CSP安全策略配置

### 测试和发布

1. **本地测试**: 使用 `npm run compile` 编译，按F5启动调试
2. **打包发布**: 使用 `vsce package` 生成.vsix文件
3. **功能验证**: 确保所有右键菜单项都能正常显示和执行

### 最新功能特性 (v4.3.0)

- ✅ 本地修订版本号显示
- ✅ SVN日志中标记未更新到本地的版本
- ✅ 优化日志界面布局，避免UI元素重叠
- ✅ 支持日志筛选和刷新功能
- ✅ 本地版本与服务器版本对比

### 开发优先级

1. **稳定性**: 确保现有功能稳定可靠
2. **用户体验**: 优化界面交互和错误提示
3. **功能完善**: 根据用户反馈添加新功能
4. **性能优化**: 提升大文件和大项目的处理性能

## 常见问题解决

1. **右键菜单不显示**: 检查 [package.json](mdc:package.json) 中的 `menus` 配置和 `when` 条件
2. **SVN命令失败**: 检查SVN工作副本路径设置和权限
3. **编码问题**: 确保设置了正确的环境变量和SVN配置
4. **WebView显示问题**: 检查CSP策略和资源路径配置

---
> Source: [summer198971/vscode-svn](https://github.com/summer198971/vscode-svn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
