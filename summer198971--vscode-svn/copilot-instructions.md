## technical-specs

> - [src/svnService.ts](mdc:src/svnService.ts) - SVN命令执行和状态管理

# VSCode SVN 插件技术规范

## 代码架构

### 核心服务层
- [src/svnService.ts](mdc:src/svnService.ts) - SVN命令执行和状态管理
- [src/aiService.ts](mdc:src/aiService.ts) - AI服务集成（OpenAI/通义千问）
- [src/commitLogStorage.ts](mdc:src/commitLogStorage.ts) - 提交日志存储管理

### 用户界面层
- [src/commitPanel.ts](mdc:src/commitPanel.ts) - 单文件提交和差异对比
- [src/folderCommitPanel.ts](mdc:src/folderCommitPanel.ts) - 文件夹批量提交
- [src/svnLogPanel.ts](mdc:src/svnLogPanel.ts) - SVN历史日志查看
- [src/updatePanel.ts](mdc:src/updatePanel.ts) - 文件更新进度显示
- [src/diffProvider.ts](mdc:src/diffProvider.ts) - 差异对比提供器

## 开发标准

### TypeScript 规范
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

### WebView 开发规范
- 使用CSP安全策略
- 消息传递使用类型化接口
- HTML模板内嵌在TypeScript文件中
- CSS样式使用内联方式，确保安全性

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

### 配置管理
- 使用 `vscode.workspace.getConfiguration('vscode-svn')` 获取配置
- 配置项命名使用驼峰式：`aiModel`, `customSvnRoot`
- 敏感信息（API密钥）使用VSCode的安全存储

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

### 菜单配置规范
- 使用 `2_svn` 组确保菜单项可见性
- `when` 条件使用 `resourceScheme == file`
- 文件和文件夹操作分别配置
- 优先级使用 `@1`, `@2` 等数字后缀

## 性能优化

### SVN命令优化
- 使用 `--xml` 参数获取结构化输出
- 设置环境变量确保UTF-8编码
- 大文件操作使用流式处理
- 缓存SVN状态信息避免重复查询

### WebView优化
- 延迟加载大量数据
- 使用虚拟滚动处理长列表
- 最小化DOM操作
- 合理使用防抖和节流

## 测试策略

### 单元测试
- 测试SVN命令解析逻辑
- 测试配置管理功能
- 测试错误处理机制

### 集成测试
- 测试完整的提交流程
- 测试AI服务集成
- 测试WebView交互

### 用户测试
- 验证右键菜单显示
- 验证中文界面显示
- 验证跨平台兼容性

## 发布流程

1. 更新版本号在 [package.json](mdc:package.json)
2. 更新功能说明在 [README.md](mdc:README.md)
3. 记录变更在 [CHANGELOG.md](mdc:CHANGELOG.md)
4. 编译: `npm run compile`
5. 打包: `vsce package`
6. 测试: 安装.vsix文件验证功能
7. 发布: `vsce publish`

---
> Source: [summer198971/vscode-svn](https://github.com/summer198971/vscode-svn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
