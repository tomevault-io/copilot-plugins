## xzlibrary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🏗️ Monorepo架构

**这个仓库采用Monorepo架构管理多个相关的Swift包。所有包都位于`Packages/`目录下：**

### 包结构

#### 1. **XZCore** (核心包 - 纯Foundation)
**位置**: `Packages/XZCore/`  
**平台支持**: iOS 15+, macOS 12+, watchOS 8+, tvOS 15+
```bash
cd Packages/XZCore && swift build  # 构建
cd Packages/XZCore && swift test   # 测试
```

**包含内容**:
- 错误处理 (`XZError`)
- 基础枚举 (`MyMethod`, `ParameterEncoding`)
- 常量定义
- Foundation扩展 (Array, String, Date, Dictionary等)
- CoreGraphics扩展 (CGFloat, 数值类型等)

#### 2. **XZNetworking** (网络包 - 跨平台)
**位置**: `Packages/XZNetworking/`  
**平台支持**: iOS 15+, macOS 12+, watchOS 8+, tvOS 15+  
**依赖**: XZCore
```bash
cd Packages/XZNetworking && swift build  # 构建
cd Packages/XZNetworking && swift test   # 测试
```

**包含内容**:
- `APIProtocol`: API协议定义
- `RequestManager`: 网络请求管理器（Combine + URLSession）
- `SSEClient`: Server-Sent Events客户端
- `NWPathMonitor+Combine`: 网络状态监控

#### 3. **XZFileUtils** (文件工具包 - 跨平台)
**位置**: `Packages/XZFileUtils/`  
**平台支持**: iOS 15+, macOS 12+, watchOS 8+, tvOS 15+  
**依赖**: XZCore
```bash
cd Packages/XZFileUtils && swift build  # 构建
cd Packages/XZFileUtils && swift test   # 测试
```

**包含内容**:
- `FileHelper`: 文件操作工具
- `FileStorage`: 文件存储管理

#### 4. **XZFoundationUtils** (Foundation工具包 - 跨平台)
**位置**: `Packages/XZFoundationUtils/`  
**平台支持**: iOS 15+, macOS 12+, watchOS 8+, tvOS 15+  
**依赖**: XZCore
```bash
cd Packages/XZFoundationUtils && swift build  # 构建
cd Packages/XZFoundationUtils && swift test   # 测试
```

**包含内容**:
- 高级Foundation扩展
- 数据处理工具
- 验证和格式化工具

#### 5. **XZUIKitUtils** (iOS UI工具包)
**位置**: `Packages/XZUIKitUtils/`  
**平台支持**: iOS 15+, tvOS 15+  
**依赖**: XZCore, XZFoundationUtils
```bash
cd Packages/XZUIKitUtils && swift build  # 构建 (仅限iOS/tvOS)
cd Packages/XZUIKitUtils && swift test   # 测试
```

**包含内容**:
- UIKit扩展 (UIView, UIButton, UIScrollView等)
- 设备检测和系统工具
- UI辅助工具

#### 6. **XZAppKitUtils** (macOS UI工具包)
**位置**: `Packages/XZAppKitUtils/`  
**平台支持**: macOS 12+  
**依赖**: XZCore, XZFoundationUtils
```bash
cd Packages/XZAppKitUtils && swift build  # 构建 (仅限macOS)
cd Packages/XZAppKitUtils && swift test   # 测试
```

**包含内容**:
- AppKit扩展
- macOS特定的UI工具

#### 7. **XZUIComponents** (UI组件包)
**位置**: `Packages/XZUIComponents/`  
**平台支持**: iOS 15+, tvOS 15+  
**依赖**: XZCore, XZUIKitUtils
```bash
cd Packages/XZUIComponents && swift build  # 构建
cd Packages/XZUIComponents && swift test   # 测试
```

**包含内容**:
- `XZProgressHUD`: 进度指示器组件
- `XZUIButton`: 自定义按钮组件
- `XZUILabel`: 自定义标签组件
- `XZFPSLabel`: FPS显示组件

## 📦 包依赖关系

```
XZCore (基础包)
├── XZNetworking (网络)
├── XZFoundationUtils (Foundation工具)
├── XZFileUtils (文件工具)
├── XZUIKitUtils (iOS UI工具)
│   └── XZUIComponents (UI组件)
└── XZAppKitUtils (macOS UI工具)
```

## 🔧 开发指南

### 选择合适的包
- **基础功能**: 仅使用 `XZCore`
- **网络请求**: 添加 `XZNetworking`
- **文件操作**: 添加 `XZFileUtils`
- **iOS UI开发**: 添加 `XZUIKitUtils` 和/或 `XZUIComponents`
- **macOS开发**: 添加 `XZAppKitUtils`
- **跨平台工具**: 添加 `XZFoundationUtils`

### 在Package.swift中使用
```swift
dependencies: [
    .package(path: "path/to/XZLibrary/Packages/XZCore"),
    .package(path: "path/to/XZLibrary/Packages/XZNetworking"),
    .package(path: "path/to/XZLibrary/Packages/XZUIKitUtils")
],
targets: [
    .target(
        name: "YourTarget",
        dependencies: ["XZCore", "XZNetworking", "XZUIKitUtils"]
    )
]
```

### 统一构建和测试命令
```bash
# 从根目录构建所有包
./Scripts/build-all.sh

# 从根目录测试所有包
./Scripts/test-all.sh
```

### 导入语句
```swift
import XZCore          // 基础功能
import XZNetworking    // 网络请求
import XZUIKitUtils    // iOS UI工具
import XZUIComponents  // UI组件
```

## ⚠️ 迁移注意事项

1. **破坏性变更**: 需要更新导入语句
2. **按需导入**: 只导入需要的包，减少编译时间
3. **平台特定**: 确保在正确的平台上使用对应的包
4. **依赖顺序**: 注意包之间的依赖关系

## 📋 开发状态

- ✅ **XZCore**: 完成迁移和测试
- ✅ **XZNetworking**: 完成迁移和测试  
- ✅ **XZFileUtils**: 完成迁移和测试
- ✅ **XZFoundationUtils**: Monorepo结构完成，包含占位代码
- ✅ **XZUIKitUtils**: Monorepo结构完成，包含占位代码
- ✅ **XZAppKitUtils**: Monorepo结构完成，包含占位代码
- ✅ **XZUIComponents**: Monorepo结构完成，包含占位代码

## 🚀 Monorepo优势

1. **统一管理**: 所有相关包在一个仓库中管理
2. **依赖简化**: 包之间使用相对路径依赖
3. **CI/CD统一**: 单个仓库的持续集成配置
4. **版本同步**: 所有包可以保持版本同步
5. **构建优化**: 提供统一的构建和测试脚本

## 📁 项目结构

```
XZLibrary/
├── .github/workflows/       # CI/CD配置
├── Packages/               # 所有Swift包
│   ├── XZCore/
│   ├── XZNetworking/
│   ├── XZFileUtils/
│   ├── XZFoundationUtils/
│   ├── XZUIKitUtils/
│   ├── XZAppKitUtils/
│   └── XZUIComponents/
├── Scripts/                # 构建和测试脚本
├── CLAUDE.md              # 本文件
└── README.md              # 项目说明

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mephsito23)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mephsito23)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
