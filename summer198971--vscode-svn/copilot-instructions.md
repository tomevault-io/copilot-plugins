## vscode-svn-plugin-development

> 这是一个专门为VSCode开发的SVN操作插件项目，提供完整的SVN版本控制功能和AI辅助提交日志生成。

# VSCode SVN 插件开发规范

## 项目概述

这是一个专门为VSCode开发的SVN操作插件项目，提供完整的SVN版本控制功能和AI辅助提交日志生成。

### 核心文件结构
- [package.json](mdc:package.json) - 插件配置、命令定义、菜单配置
- [src/extension.ts](mdc:src/extension.ts) - 插件主入口，注册所有命令和功能
- [src/svnService.ts](mdc:src/svnService.ts) - SVN操作核心服务类
- [src/svnLogPanel.ts](mdc:src/svnLogPanel.ts) - SVN日志查看面板，支持版本对比和筛选
- [src/commitPanel.ts](mdc:src/commitPanel.ts) - 单文件提交和差异对比面板
- [src/folderCommitPanel.ts](mdc:src/folderCommitPanel.ts) - 文件夹批量提交面板
- [src/aiService.ts](mdc:src/aiService.ts) - AI提交日志生成服务
- [README.md](mdc:README.md) - 功能文档和使用说明
- [CHANGELOG.md](mdc:CHANGELOG.md) - 版本更新记录

## 开发核心原则

### 1. 中文优先原则
- **所有代码注释必须使用中文**
- **所有用户界面文本必须使用中文**
- **错误信息和提示信息必须使用中文**
- **工具提示(tooltip)使用简洁的中文说明**

### 2. 用户体验优先
- **确保所有SVN操作命令都出现在右键上下文菜单中**
- **优先考虑视觉表现和用户友好体验**
- **提供清晰的视觉反馈和状态指示**
- **错误状态要有明确的提示和解决方案**

### 3. 严格按需求开发
- **避免过度创新，严格按照需求开发**
- **每次修改都要仔细执行，避免破坏相关逻辑**
- **功能实现完成后不要自动升级版本号**
- **版本管理完全由用户决定**

## TypeScript 开发规范

### 代码风格
```typescript
// 接口定义示例
interface SvnLogEntry {
    revision: string;
    author: string;
    date: string;
    message: string;
    isNewerThanLocal?: boolean; // 可选属性用?标记
}

// 类定义示例
export class SvnService {
    private _workingCopyPath: string | undefined;
    
    // 私有方法用_前缀
    private async _executeCommand(command: string): Promise<string> {
        // 实现
    }
    
    // 公共方法
    public async getStatus(filePath: string): Promise<SvnStatus> {
        // 实现
    }
}
```

### 错误处理标准
```typescript
try {
    const result = await svnService.executeCommand(command);
    return result;
} catch (error) {
    vscode.window.showErrorMessage(`SVN操作失败: ${error.message}`);
    throw error;
}
```

### 命令注册模式
```typescript
// 在extension.ts中注册命令
const disposable = vscode.commands.registerCommand('vscode-svn.commandName', async (uri: vscode.Uri) => {
    try {
        // 命令实现
    } catch (error) {
        vscode.window.showErrorMessage(`操作失败: ${error.message}`);
    }
});
context.subscriptions.push(disposable);
```

## WebView 开发规范

### 安全策略
- 使用CSP安全策略
- 消息传递使用类型化接口
- HTML模板内嵌在TypeScript文件中
- CSS样式使用内联方式，确保安全性

### 样式规范
```css
.container {
    padding: 20px;
    font-family: var(--vscode-font-family);
    color: var(--vscode-foreground);
    background-color: var(--vscode-editor-background);
}

.button {
    background-color: var(--vscode-button-background);
    color: var(--vscode-button-foreground);
    border: none;
    padding: 8px 16px;
    border-radius: 2px;
    cursor: pointer;
}
```

### 状态指示器颜色
- **修改 (Modified)**: `var(--vscode-gitDecoration-modifiedResourceForeground)` - 橙色
- **新增 (Added)**: `var(--vscode-gitDecoration-addedResourceForeground)` - 绿色  
- **删除 (Deleted)**: `var(--vscode-gitDecoration-deletedResourceForeground)` - 红色
- **冲突 (Conflict)**: `var(--vscode-gitDecoration-conflictingResourceForeground)` - 红色

## 配置管理规范

### package.json 配置要点
- `commands`: 定义所有SVN操作命令
- `menus`: 配置右键上下文菜单，使用 `2_svn` 组确保菜单项可见性
- `configuration`: 定义用户可配置的设置项
- `when` 条件使用 `resourceScheme == file`

### 重要配置项
- `vscode-svn.customSvnRoot`: 自定义SVN工作副本路径
- `vscode-svn.aiModel`: AI模型选择 (openai/qwen)
- `vscode-svn.aiApiKey`: OpenAI API密钥
- `vscode-svn.qwenApiKey`: 通义千问API密钥

## SVN 操作优化

### 命令执行优化
- 使用 `--xml` 参数获取结构化输出
- 设置环境变量确保UTF-8编码
- 大文件操作使用流式处理
- 缓存SVN状态信息避免重复查询

### 性能优化策略
- 延迟加载大量数据
- 使用虚拟滚动处理长列表
- 最小化DOM操作
- 合理使用防抖和节流

## 功能模块架构

### 核心服务层
- [src/svnService.ts](mdc:src/svnService.ts) - SVN命令执行和状态管理
- [src/aiService.ts](mdc:src/aiService.ts) - AI服务集成（OpenAI/通义千问）
- [src/commitLogStorage.ts](mdc:src/commitLogStorage.ts) - 提交日志存储管理

### 用户界面层
- [src/commitPanel.ts](mdc:src/commitPanel.ts) - 单文件提交和差异对比
- [src/folderCommitPanel.ts](mdc:src/folderCommitPanel.ts) - 文件夹批量提交
- [src/svnLogPanel.ts](mdc:src/svnLogPanel.ts) - SVN历史日志查看，支持筛选和版本对比
- [src/updatePanel.ts](mdc:src/updatePanel.ts) - 文件更新进度显示
- [src/diffProvider.ts](mdc:src/diffProvider.ts) - 差异对比提供器

## 开发注意事项

### 版本管理
- **不要自动升级版本号**
- **版本管理完全由用户决定**
- **只有用户明确要求时才更新版本和文档**

### 菜单配置
- 新增命令时必须同时配置在 `explorer/context` 和 `editor/context` 菜单中
- 使用 `@1`, `@2` 等数字后缀设置优先级

### 编码问题
- 特别注意SVN操作中的中文编码问题
- 设置UTF-8环境变量
- 处理跨平台兼容性

### 调试和日志
- 使用输出通道记录详细日志
- 提供调试信息帮助问题排查
- 错误信息要用户友好且具体

## 测试和发布流程

### 本地测试
1. 使用 `npm run compile` 编译
2. 按F5启动调试
3. 验证所有右键菜单项显示和执行

### 功能验证清单
- [ ] 所有文本都是中文
- [ ] 右键菜单显示完整
- [ ] 在不同主题下显示正常
- [ ] 错误提示友好
- [ ] 跨平台兼容性

## 当前版本特性

### 最新功能 (基于代码分析)
- ✅ SVN日志查看和版本对比
- ✅ 本地修订版本号显示
- ✅ 日志筛选功能（按版本、日期、作者、内容）
- ✅ 筛选数量提示和状态显示
- ✅ 文件差异对比查看
- ✅ AI提交日志生成
- ✅ 批量文件提交管理

### 界面优化
- 筛选状态用橙色显示，便于区分
- 日志列表头部显示详细筛选信息
- 工具栏显示当前记录数量
- 支持"只显示相关文件"过滤

## 开发优先级

1. **稳定性**: 确保现有功能稳定可靠
2. **用户体验**: 优化界面交互和错误提示
3. **功能完善**: 根据用户反馈添加新功能
4. **性能优化**: 提升大文件和大项目的处理性能

---
> Source: [summer198971/vscode-svn](https://github.com/summer198971/vscode-svn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
