## auto-get-layer

> const selection = figma.currentPage.selection;

# Figma API 使用模式和开发工作流

## Figma Plugin API 关键模式

### 节点选择和操作
```typescript
// 获取当前选中的节点
const selection = figma.currentPage.selection;

// 节点定位和视图控制
figma.currentPage.selection = [node];
figma.viewport.scrollAndZoomIntoView([node]);
```

### 节点类型判断
基于 [code.ts](mdc:code.ts) 中定义的常量进行类型检查：

```typescript
// 检查节点是否具有 children 属性
const NODE_TYPES_WITH_CHILDREN = [
  "DOCUMENT", "PAGE", "FRAME", "GROUP", "COMPONENT", 
  "COMPONENT_SET", "INSTANCE", "SECTION"
];

const hasChildren = NODE_TYPES_WITH_CHILDREN.includes(node.type);

// 原子节点类型（没有 children）
const ATOMIC_NODE_TYPES = [
  "RECTANGLE", "LINE", "ELLIPSE", "POLYGON", "STAR", 
  "VECTOR", "TEXT", "SLICE", "BOOLEAN_OPERATION"
];
```

### 递归节点遍历模式
```typescript
function analyzeNodeRecursively(node: SceneNode, depth = 0, rootAncestor = null) {
  // 处理当前节点
  
  // 递归处理子节点
  if ('children' in node && node.children) {
    for (const child of node.children) {
      analyzeNodeRecursively(child, depth + 1, rootAncestor || node);
    }
  }
}
```

## 主线程与UI线程通信

### 主线程向UI发送消息
```typescript
// 在 code.ts 中
figma.ui.postMessage({
  type: 'nodeInfo',
  data: nodeInfo
});

figma.ui.postMessage({
  type: 'checkResults', 
  data: checkResults
});
```

### UI线程向主线程发送消息
```typescript
// 在 ui.html 中
parent.postMessage({
  pluginMessage: {
    type: 'analyzeSelection'
  }
}, '*');

parent.postMessage({
  pluginMessage: {
    type: 'downloadImages',
    nodes: selectedNodes
  }
}, '*');
```

## 图片导出模式

### 节点图片导出
```typescript
// PNG 格式导出，2x 缩放
const imageData = await node.exportAsync({
  format: 'PNG',
  constraint: { type: 'SCALE', value: 2 }
});
```

### 批量图片处理
导出的图片数据通过 UI 线程进行下载处理，参考 [ui.html](mdc:ui.html) 中的实现。

## 节点属性提取模式

### 基础属性提取
```typescript
const basicInfo = {
  id: node.id,
  name: node.name,
  type: node.type,
  visible: node.visible,
  locked: node.locked
};
```

### 几何属性提取
```typescript
const geometryInfo = {
  x: node.x,
  y: node.y, 
  width: node.width,
  height: node.height,
  rotation: node.rotation
};
```

### 样式属性提取
根据节点类型提取相应的样式属性（fills, strokes, effects 等）

## 开发工作流

### 1. 开发环境启动
使用 [start-dev.sh](mdc:start-dev.sh) 脚本启动开发环境：
```bash
./start-dev.sh
```

### 2. TypeScript 编译
项目使用 [tsconfig.json](mdc:tsconfig.json) 配置进行 TypeScript 编译

### 3. 插件调试
- 在 Figma 中加载插件进行测试
- 使用 `console.log()` 在开发者工具中查看日志
- UI 相关调试在浏览器开发者工具中进行

### 4. 错误处理模式
```typescript
try {
  // 可能失败的 Figma API 操作
  const result = await someOperation();
} catch (error) {
  console.error('操作失败:', error);
  figma.ui.postMessage({
    type: 'error',
    message: '操作失败，请重试'
  });
}
```

## 性能优化建议

### 1. 避免重复遍历
- 一次遍历收集所有需要的信息
- 缓存计算结果避免重复计算

### 2. 限制递归深度
- 设置合理的递归深度限制
- 对于大型设计文件进行分批处理

### 3. 异步操作优化
- 使用 `await` 正确处理异步 API
- 批量导出时控制并发数量

## 常见问题和解决方案

### 1. 节点类型兼容性
- 始终检查节点是否具有特定属性再使用
- 使用类型守卫确保代码安全

### 2. UI 状态同步
- 通过消息机制保持主线程和UI线程的状态同步
- 适当的加载状态提示

### 3. 内存管理
- 及时清理不需要的引用
- 避免在长时间运行中积累过多数据

## 插件配置

### manifest.json 配置要点
参考 [manifest.json](mdc:manifest.json)：
- 正确设置 API 版本
- 配置所需的权限
- 设置合适的插件尺寸

### package.json 依赖管理
参考 [package.json](mdc:package.json) 中的依赖配置和脚本设置

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tianmuji)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tianmuji)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
