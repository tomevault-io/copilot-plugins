## ui-ux-guidelines

> - 错误信息、提示信息、按钮文字都要中文化

# VSCode SVN 插件 UI/UX 设计规范

## 设计原则

### 1. 中文优先
- 所有用户界面文本必须使用中文
- 错误信息、提示信息、按钮文字都要中文化
- 工具提示(tooltip)使用简洁的中文说明

### 2. 视觉一致性
- 遵循VSCode的设计语言和主题
- 使用VSCode内置的图标和颜色变量
- 保持与VSCode原生界面的一致性

### 3. 用户友好
- 操作流程要直观明了
- 提供清晰的视觉反馈
- 错误状态要有明确的提示和解决方案

## 界面组件规范

### WebView 面板设计

#### 布局结构
```css
.container {
    padding: 20px;
    font-family: var(--vscode-font-family);
    color: var(--vscode-foreground);
    background-color: var(--vscode-editor-background);
}

.header {
    border-bottom: 1px solid var(--vscode-panel-border);
    padding-bottom: 10px;
    margin-bottom: 20px;
}

.content {
    display: flex;
    gap: 20px;
}
```

#### 按钮样式
```css
.button {
    background-color: var(--vscode-button-background);
    color: var(--vscode-button-foreground);
    border: none;
    padding: 8px 16px;
    border-radius: 2px;
    cursor: pointer;
    font-size: 13px;
}

.button:hover {
    background-color: var(--vscode-button-hoverBackground);
}

.button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}
```

### 状态指示器

#### 文件状态颜色
- **修改 (Modified)**: `var(--vscode-gitDecoration-modifiedResourceForeground)` - 橙色
- **新增 (Added)**: `var(--vscode-gitDecoration-addedResourceForeground)` - 绿色  
- **删除 (Deleted)**: `var(--vscode-gitDecoration-deletedResourceForeground)` - 红色
- **冲突 (Conflict)**: `var(--vscode-gitDecoration-conflictingResourceForeground)` - 红色

#### 版本对比标记
```css
.newer-version {
    border-left: 3px solid var(--vscode-notificationsWarningIcon-foreground);
    background-color: var(--vscode-inputValidation-warningBackground);
}

.newer-badge {
    background-color: var(--vscode-badge-background);
    color: var(--vscode-badge-foreground);
    padding: 2px 6px;
    border-radius: 10px;
    font-size: 11px;
    margin-left: 8px;
}
```

## 交互设计规范

### 右键菜单
- 菜单项按功能分组，使用 `2_svn` 组
- 常用操作放在前面：提交、更新、查看日志
- 管理操作放在后面：设置SVN路径
- 菜单项文字要简洁明了

### 对话框设计
- 使用VSCode原生的 `showInputBox`, `showQuickPick` 等API
- 提供默认值和占位符文本
- 验证用户输入并提供即时反馈

### 进度指示
```typescript
// 长时间操作使用进度条
vscode.window.withProgress({
    location: vscode.ProgressLocation.Notification,
    title: "正在执行SVN操作...",
    cancellable: true
}, async (progress, token) => {
    // 操作实现
});
```

## 响应式设计

### WebView 自适应
```css
@media (max-width: 800px) {
    .content {
        flex-direction: column;
    }
    
    .sidebar {
        width: 100%;
        margin-bottom: 20px;
    }
}
```

### 文字大小适配
- 使用相对单位 `em` 或 `rem`
- 遵循用户的字体大小设置
- 确保在不同缩放级别下的可读性

## 错误处理界面

### 错误信息显示
```typescript
// 友好的错误提示
vscode.window.showErrorMessage(
    "SVN操作失败：工作副本路径未设置", 
    "设置路径", 
    "查看帮助"
).then(selection => {
    if (selection === "设置路径") {
        vscode.commands.executeCommand('vscode-svn.setSvnRoot');
    }
});
```

### 空状态设计
```html
<div class="empty-state">
    <div class="empty-icon">📁</div>
    <h3>暂无SVN日志</h3>
    <p>该文件或文件夹还没有SVN提交记录</p>
    <button onclick="refresh()">刷新</button>
</div>
```

## 可访问性

### 键盘导航
- 所有交互元素支持Tab键导航
- 提供快捷键支持
- 焦点状态要清晰可见

### 屏幕阅读器支持
```html
<button aria-label="提交当前文件到SVN" title="提交文件">
    提交
</button>
```

## 性能优化

### 界面渲染优化
- 大列表使用虚拟滚动
- 图片懒加载
- 防抖处理用户输入

### 数据加载策略
- 分页加载历史记录
- 缓存常用数据
- 显示加载状态

## 测试检查清单

### 界面测试
- [ ] 所有文本都是中文
- [ ] 在不同主题下显示正常
- [ ] 响应式布局工作正常
- [ ] 按钮状态正确显示

### 交互测试  
- [ ] 右键菜单显示完整
- [ ] 键盘导航流畅
- [ ] 错误提示友好
- [ ] 进度指示准确

### 兼容性测试
- [ ] Windows/Mac/Linux 显示一致
- [ ] 不同VSCode版本兼容
- [ ] 高DPI屏幕显示清晰
- [ ] 深色/浅色主题适配

---
> Source: [summer198971/vscode-svn](https://github.com/summer198971/vscode-svn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
