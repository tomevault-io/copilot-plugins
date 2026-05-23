## dev-tools

> 此文件为在此代码库中工作的 Claude Code (claude.ai/code) 提供指导。

# CLAUDE.md

此文件为在此代码库中工作的 Claude Code (claude.ai/code) 提供指导。

## 项目概述

这是一个基于 Wails v2 + Vue 3 + TypeScript 构建的桌面开发者工具应用。它提供了一系列常用的开发者实用工具，如 JSON/XML 编辑器、时间转换器、URL 工具、Base64 转换、二维码生成、颜色转换等。

## 架构

**后端 (Go):**
- `main.go` - 入口点，包含 Wails 应用配置
- `app.go` - 主应用结构体和基本方法
- `json_processor.go` - JSON 处理工具
- `xml_processor.go` - XML 处理工具
- 使用 Wails v2 作为桌面应用框架

**前端 (Vue 3):**
- `frontend/src/App.vue` - 主 Vue 应用组件
- `frontend/src/views/` - 各个工具视图 (JsonEditor, TimeConverter, ColorConverter 等)
- `frontend/src/components/` - 可复用的 Vue 组件
- `frontend/src/stores/` - Pinia 状态管理
- `frontend/src/router/` - Vue Router 配置
- 使用 Element Plus 作为 UI 组件库
- 使用 Monaco Editor 进行代码编辑
- 使用 Vite 作为构建工具

## 常用命令

### 开发
```bash
# 启动开发服务器（热重载）
wails dev

# 安装前端依赖
cd frontend && npm install

# 安装 Go 依赖
go mod tidy
```

### 构建
```bash
# 为当前平台构建
wails build

# 为 macOS 通用版构建
make build-mac
# 或
wails build -platform=darwin/universal

# 为 Windows 构建
make build-win
# 或
wails build -platform=windows/amd64

# 带调试/开发工具构建
make build-mac-dev
make build-win-dev
```

### 前端开发
```bash
cd frontend

# 开发服务器
npm run dev

# 仅构建前端
npm run build

# 类型检查
vue-tsc --noEmit
```

### macOS 分发
```bash
# 创建 DMG 包
make dmg

# 清理构建产物
make clean
```

## 项目结构

- 后端 Go 文件位于根目录
- 前端 Vue 应用位于 `frontend/` 目录
- Wails 配置文件 `wails.json`
- 构建输出至 `build/bin/`
- 文档和图片位于 `docs/`

## 关键依赖

**Go:**
- `github.com/wailsapp/wails/v2` - 桌面应用框架
- `github.com/gopherjs/gopherjs` - JS 互操作工具

**前端:**
- Vue 3 + TypeScript
- Element Plus (UI 组件)
- Monaco Editor (代码编辑)
- Pinia (状态管理)
- 各种特定工具的实用库

## 开发注意事项

- 应用使用嵌入式资源 (`//go:embed all:frontend/dist`)
- 前端通过 Wails 绑定与 Go 后端通信
- 多个工具视图组织为独立的 Vue 组件
- 工具状态和偏好设置通过 Pinia 存储管理
- 应用支持拖拽重排序工具卡片
- 所有工具都支持实时处理和预览

## UI 设计规范

### 统一的模块布局风格

所有工具模块都应该采用统一的布局风格，参考 `Base64Text.vue` 作为标准模板：

**顶部导航栏结构：**
```vue
<div class="top-header">
  <div class="tab-nav">
    <!-- 左侧：功能按钮/标签页 -->
    <button class="tab-btn" :class="{ active: condition }">标签</button>
    <button class="action-btn">示例</button>
    <!-- 其他控件如下拉框 -->
  </div>
  <div class="tab-actions">
    <!-- 右侧：清空按钮 -->
    <button class="clear-btn" title="清空">× 清空</button>
  </div>
</div>
```

**内容区域结构：**
```vue
<div class="content-layout">
  <div class="input-panel">
    <textarea class="text-area" />
  </div>
  <div class="output-panel">
    <textarea class="text-area" />
  </div>
</div>
```

### 样式规范

**顶部导航栏样式（重要）：**
```css
.top-header {
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  padding: 0;
  margin: 0;
  background: #f8f9fa;  /* 统一背景色 */
  height: 28px;         /* 统一高度 */
  flex-shrink: 0;
}

.tab-nav {
  display: flex;
  align-items: stretch;
  background: #f8f9fa;   /* 统一背景色 */
}

.tab-actions {
  display: flex;
  align-items: stretch;  /* 改为 stretch，不是 center */
  background: #f8f9fa;   /* 统一背景色，不是白色 */
}
```

**按钮样式（必须严格遵循）：**
- 统一高度：`height: 100%` （继承父容器28px）
- 字体大小：`font-size: 10px`
- 颜色：`color: #6c757d`
- 背景：`background: #f8f9fa`
- 圆角：`border-radius: 0` （无圆角）
- 内边距：`padding: 0 10px`
- 最小宽度：`min-width: 45px`
- hover 状态：`background: #e9ecef`
- active 状态：`background: #ffffff; color: #212529`
- **边框规则**：
  - 第一个按钮：`border-left: 1px solid #d1d5db`
  - 相邻按钮：`border-left: 1px solid #d1d5db`（避免重复）
  - 最后一个按钮：`border-right: 1px solid #d1d5db`
  - 清空按钮：`border-left: 1px solid #d1d5db; border-right: 1px solid #d1d5db; border-top: none; border-bottom: none`

**按钮边框完整示例：**
```css
/* 标签页按钮组 */
.tab-btn:first-child {
  border-left: 1px solid #d1d5db;
}
.tab-btn + .tab-btn {
  border-left: 1px solid #d1d5db;
}
.tab-btn:last-child {
  border-right: 1px solid #d1d5db;
}

/* 示例按钮 */
.action-btn {
  border-left: 1px solid #d1d5db;
  border-right: 1px solid #d1d5db;
}

/* 清空按钮 */
.clear-btn {
  border-top: none;
  border-bottom: none;
  border-left: 1px solid #d1d5db;
  border-right: 1px solid #d1d5db;
}
```

**文本区域样式：**
- 字体：`font-family: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace`
- 字体大小：`font-size: 11px`
- 行高：`line-height: 1.4`
- 颜色：`color: #212529`
- 内边距：`padding: 12px`
- 占位符颜色：`color: #9ca3af`

**布局样式：**
- 容器内边距：`padding: 16px`
- 顶部导航栏：`height: 28px; margin-bottom: 16px`
- 左右面板间距：`gap: 16px`
- 边框：`border: 1px solid #d1d5db`
- 背景：`background: #ffffff`

**响应式断点：**
- 桌面端：默认左右布局
- 移动端：`@media (max-width: 768px)` 改为上下布局

### 功能规范

**自动处理：**
- 输入变化时自动处理/转换，不需要手动点击转换按钮
- 提供示例按钮，点击加载预设示例数据
- 清空按钮清除所有输入输出内容
- 从其他页面切换回来时，如果有内容应自动处理

**Monaco Editor 配置：**
```javascript
const editorOptions = {
  fontSize: 11,
  tabSize: 2,
  minimap: { enabled: false },
  scrollBeyondLastLine: true,
  automaticLayout: true,
  wordWrap: 'on',
  lineNumbers: 'on',
  readOnly: true // 输出区域设为只读
}
```

### 重要提醒

- **新增工具模块时必须遵循以上设计规范**
- **所有按钮、文本框、布局都要与现有模块保持一致**
- **特别注意按钮边框规则，确保按钮之间有正确的分隔线**
- **顶部导航栏背景必须统一为 `#f8f9fa`**
- **避免添加不必要的 UI 元素，保持界面简洁**
- **优先考虑自动化处理，减少用户手动操作**

### 开发新模块时的检查清单

创建新工具模块时，请确保：

1. ✅ 顶部导航栏背景色为 `#f8f9fa`
2. ✅ 所有按钮高度为 `100%`，字体大小为 `10px`
3. ✅ 按钮边框按规则设置，确保有正确的分隔线
4. ✅ 清空按钮样式正确（左右边框，上下无边框）
5. ✅ hover效果统一为 `#e9ecef`
6. ✅ 容器使用 `margin: 0 !important; padding: 0 !important`
7. ✅ 文本区域使用等宽字体和统一的字体大小
8. ✅ 支持自动处理和示例数据加载

## 文件保存功能

### 后端实现 (app.go)

使用通用的 `SaveFile` 方法，支持保存任何类型的文件：

```go
// 文件过滤器类型
type FileFilter struct {
	DisplayName string `json:"displayName"`
	Pattern     string `json:"pattern"`
}

// 保存文件选项
type SaveFileOptions struct {
	Title           string       `json:"title"`
	DefaultFilename string       `json:"defaultFilename"`
	Filters         []FileFilter `json:"filters"`
}

// 通用保存文件方法
func (a *App) SaveFile(content string, options SaveFileOptions, isBase64 bool) (string, error) {
	// 转换过滤器格式
	var filters []runtime.FileFilter
	for _, filter := range options.Filters {
		filters = append(filters, runtime.FileFilter{
			DisplayName: filter.DisplayName,
			Pattern:     filter.Pattern,
		})
	}

	// 弹出原生保存对话框
	fileName, err := runtime.SaveFileDialog(a.ctx, runtime.SaveDialogOptions{
		DefaultFilename: options.DefaultFilename,
		Title:          options.Title,
		Filters:        filters,
	})
	
	if err != nil || fileName == "" {
		return "", fmt.Errorf("用户取消保存")
	}

	var data []byte
	if isBase64 {
		// 解码base64数据
		data, err = base64.StdEncoding.DecodeString(content)
		if err != nil {
			return "", fmt.Errorf("解码base64数据失败: %v", err)
		}
	} else {
		// 普通文本
		data = []byte(content)
	}

	// 写入文件
	err = os.WriteFile(fileName, data, 0644)
	if err != nil {
		return "", fmt.Errorf("写入文件失败: %v", err)
	}

	return fileName, nil
}
```

### 前端工具库 (frontend/src/utils/fileUtils.ts)

提供完整的文件保存工具库：

```typescript
import { SaveFile } from '../../wailsjs/go/main/App'
import { main } from '../../wailsjs/go/models'
import { ElMessage } from 'element-plus'

// 常用文件过滤器
export const FILE_FILTERS = {
  IMAGE_PNG: { displayName: 'PNG图片 (*.png)', pattern: '*.png' },
  IMAGE_JPG: { displayName: 'JPEG图片 (*.jpg)', pattern: '*.jpg' },
  TEXT: { displayName: '文本文件 (*.txt)', pattern: '*.txt' },
  JSON: { displayName: 'JSON文件 (*.json)', pattern: '*.json' },
  XML: { displayName: 'XML文件 (*.xml)', pattern: '*.xml' },
  CSV: { displayName: 'CSV文件 (*.csv)', pattern: '*.csv' },
  ALL: { displayName: '所有文件 (*.*)', pattern: '*.*' }
}

// 保存图片（base64格式）
export async function saveImage(
  base64Data: string,
  filename: string = 'image.png',
  title: string = '保存图片'
): Promise<string> {
  const filters = [
    new main.FileFilter(FILE_FILTERS.IMAGE_PNG),
    new main.FileFilter(FILE_FILTERS.ALL)
  ]
  
  const options = new main.SaveFileOptions({
    title,
    defaultFilename: filename,
    filters
  })

  const savedPath = await SaveFile(base64Data, options, true)
  return savedPath
}

// 保存文本文件
export async function saveText(
  content: string,
  filename: string = 'file.txt',
  title: string = '保存文件',
  customFilters: any[] = [FILE_FILTERS.TEXT, FILE_FILTERS.ALL]
): Promise<string> {
  const filters = customFilters.map(filter => new main.FileFilter(filter))
  
  const options = new main.SaveFileOptions({
    title,
    defaultFilename: filename,
    filters
  })

  const savedPath = await SaveFile(content, options, false)
  return savedPath
}

// 保存JSON文件
export async function saveJSON(data: any, filename: string = 'data.json'): Promise<string> {
  const content = JSON.stringify(data, null, 2)
  return saveText(content, filename, '保存JSON文件', [FILE_FILTERS.JSON, FILE_FILTERS.ALL])
}
```

### 使用方法

在任何 Vue 组件中：

```typescript
import { saveImage, saveText, saveJSON } from '../utils/fileUtils'
import { ElMessage } from 'element-plus'

// 保存图片
try {
  const savedPath = await saveImage(base64Data, 'qrcode.png', '保存二维码图片')
  ElMessage.success(`图片已保存到: ${savedPath}`)
} catch (error) {
  if (!error?.toString().includes('用户取消保存')) {
    ElMessage.error(`保存失败: ${error}`)
  }
}

// 保存文本
try {
  const savedPath = await saveText(textContent, 'output.txt', '保存处理结果')
  ElMessage.success(`文件已保存到: ${savedPath}`)
} catch (error) {
  ElMessage.error(`保存失败: ${error}`)
}
```

### 注意事项

1. **类型安全**：必须使用 Wails 生成的类型 `main.FileFilter` 和 `main.SaveFileOptions`
2. **重新生成绑定**：修改后端方法后需要运行 `wails generate module` 重新生成前端绑定
3. **UI 提示分离**：工具函数不包含 UI 提示，由调用方决定如何处理成功和错误
4. **用户取消处理**：用户取消保存时不显示错误提示，这是正常操作
5. **原生对话框**：使用系统原生的保存对话框，用户体验更好

## 应用更新检查功能

应用支持自动检查更新，采用双重降级策略：优先使用自定义服务器，失败时自动降级到 GitHub API。

### 后端实现 (app.go)

**主要方法：**

```go
// CheckForUpdate 检查更新（优先使用自定义服务器，失败时降级到 GitHub API）
func (a *App) CheckForUpdate(owner, repo string) (*UpdateInfo, error) {
	// 优先尝试从自定义服务器获取
	customURL := "http://gotool.51godream.com/version.json"
	updateInfo, err := a.checkCustomServer(customURL)
	if err == nil {
		return updateInfo, nil
	}

	// 自定义服务器失败，降级到 GitHub API
	return a.checkGitHubAPI(owner, repo)
}
```

**工作流程：**
1. 首先尝试从 `http://gotool.51godream.com/version.json` 获取版本信息（5秒超时）
2. 如果自定义服务器失败（超时、404、JSON解析错误等），自动降级到 GitHub API
3. GitHub API 作为备用方案（10秒超时）

### 自定义服务器 version.json 格式

在服务器上部署 `version.json` 文件，格式如下：

```json
{
  "version": "v0.2.0",
  "description": "## 更新内容\n\n### 新功能\n- 添加了颜色转换器\n- 添加了检查更新功能\n\n### 改进\n- 优化了窗口大小记忆功能\n- 改进了UI样式\n\n### 修复\n- 修复了拖拽卡片的问题\n- 修复了窗口控制按钮的问题",
  "downloadUrls": {
    "darwin": "http://gotool.51godream.com/downloads/dev-tools-v0.2.0-darwin.dmg",
    "windows": "http://gotool.51godream.com/downloads/dev-tools-v0.2.0-windows.exe"
  }
}
```

**字段说明：**
- `version`: 版本号（建议带 v 前缀，如 v0.2.0）
- `description`: 更新说明（支持 Markdown 格式）
- `downloadUrls`: 多平台下载地址对象
  - `darwin`: macOS 平台的下载地址（自动检测）
  - `windows`: Windows 平台的下载地址（自动检测）
  - 应用会根据当前运行平台（通过 `runtime.GOOS`）自动选择对应的下载地址
  - 如果没有配置对应平台的地址，前端会自动降级到 GitHub releases 页面

### 前端使用 (MainLayout.vue)

```typescript
const checkForUpdate = async () => {
  const loading = ElMessage({
    message: '正在检查更新...',
    type: 'info',
    duration: 0
  });

  try {
    const owner = 'zzdylan';
    const repo = 'dev-tools';

    const updateInfo = await CheckForUpdate(owner, repo);
    loading.close();

    if (updateInfo.hasUpdate) {
      // 显示更新对话框
      await ElMessageBox.confirm(
        `发现新版本: ${updateInfo.latestVersion}...`,
        '发现新版本',
        { type: 'info' }
      );
      // 打开下载页面
      window.open(updateInfo.downloadUrl, '_blank');
    } else {
      ElMessage.success('当前已是最新版本');
    }
  } catch (error) {
    loading.close();
    ElMessage.error(`检查更新失败: ${error}`);
  }
};
```

### 版本比较规则

使用语义化版本比较：
- 支持 `v1.2.3` 和 `1.2.3` 格式
- 逐段比较主版本号、次版本号、修订号
- 缺失的版本段视为 0

### 注意事项

1. **自定义服务器优先**：确保自定义服务器响应速度快（5秒超时）
2. **降级策略**：服务器失败时自动使用 GitHub API，无需人工干预
3. **协议支持**：自定义服务器使用 HTTP，GitHub API 使用 HTTPS
4. **跨域配置**：自定义服务器需要正确配置 CORS 头
5. **版本格式**：建议统一使用 `v` 前缀的版本号
6. **日志输出**：降级时会在控制台输出日志，便于调试

## 工具模块

### 颜色转换器 (ColorConverter.vue)

颜色转换器提供十六进制(HEX)和RGB颜色格式之间的相互转换功能。

**主要功能：**
- 十六进制 ↔ RGB 颜色格式互转
- 实时颜色预览，支持自动对比色文字显示
- 示例按钮加载预设颜色
- 清空按钮重置所有输入

**技术实现：**
- 使用 Pinia 状态管理保存颜色数据
- 自动转换：输入任一格式时自动计算其他格式
- 响应式布局：桌面端左右布局，移动端上下布局
- 完整的颜色转换算法实现

**状态管理：**
```typescript
// stores/tools.ts
colorConverter: {
  hexColor: '#FF5733',
  rgbColor: { r: 255, g: 87, b: 51 }
}
```

**布局规范：**
- 严格遵循项目统一UI设计规范
- 顶部导航栏高度28px，按钮边框规则一致
- 移动端布局：输入区域200px高度，颜色预览150px高度
- 颜色预览区域设置合理的padding确保边框完整显示

## 菜单管理规则

### 工具排序规范

工具在左侧导航菜单中的显示顺序应遵循以下原则：

1. **核心工具优先**：JSON/XML编辑器等核心功能排在前面
2. **通用工具居中**：时间转换、URL工具、编码工具等常用功能
3. **特殊工具靠后**：特定用途或敏感工具排在后面

### 默认显示规则

- **常规工具**：默认 `visible: true`，用户可以直接使用
- **敏感工具**：默认 `visible: false`，需要用户在设置中手动启用
  - 示例：Charles激活码生成器等破解/激活相关工具

### 配置位置

菜单配置位于 `frontend/src/stores/tools.ts` 中的两个位置：
1. `menuConfig.items` - 主要配置
2. `initializeMenu()` 方法中的 `defaultItems` - 默认项配置

**重要**：两个位置的配置必须保持同步！

### 配置示例

```typescript
// 常规工具
{ id: 'color', path: '/color-converter', icon: '🎨', title: '颜色转换器', visible: true, order: 14, description: '十六进制、RGB颜色格式转换' }

// 敏感工具
{ id: 'charles', path: '/charles-generator', icon: '🔑', title: 'Charles 激活码', visible: false, order: 15, description: '生成Charles代理工具的激活码' }
```

---
> Source: [zzdylan/dev-tools](https://github.com/zzdylan/dev-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
