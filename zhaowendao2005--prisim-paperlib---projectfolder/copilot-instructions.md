## prisim-paperlib-projectfolder

> 项目使用 **CSS 变量** 实现主题切换，所有颜色、阴影等视觉属性必须通过变量引用，禁止硬编码颜色值。

# 主题系统规范

## 概述

项目使用 **CSS 变量** 实现主题切换，所有颜色、阴影等视觉属性必须通过变量引用，禁止硬编码颜色值。

## 目录结构

```
apps/client/src/renderer/styles/
├── index.css              # 入口，导入 Tailwind 和主题
└── themes/
    └── variables.css      # 主题变量定义
```

## 主题切换机制

通过在根元素添加 `.dark` 类切换主题：

```vue
<div class="app-container" :class="{ dark: isDark }">
```

## 变量命名规范

### 命名格式

```
--[类型]-[用途]-[状态?]
```

| 前缀 | 含义 | 示例 |
|------|------|------|
| `--color-bg-` | 背景色 | `--color-bg-primary` |
| `--color-text-` | 文字色 | `--color-text-muted` |
| `--color-border` | 边框色 | `--color-border-light` |
| `--color-accent` | 强调色 | `--color-accent-hover` |
| `--shadow-` | 阴影 | `--shadow-lg` |
| `--traffic-` | 交通灯按钮 | `--traffic-close` |

### 完整变量列表

#### 背景色
| 变量 | 用途 |
|------|------|
| `--color-bg-primary` | 页面主背景 |
| `--color-bg-secondary` | 卡片、面板背景 |
| `--color-bg-sidebar` | 侧边栏背景（带透明度） |
| `--color-bg-card` | 卡片背景 |
| `--color-bg-input` | 输入框背景 |
| `--color-bg-hover` | 悬停状态背景 |
| `--color-bg-active` | 激活状态背景 |
| `--color-bg-toolbar` | 工具栏背景（带透明度） |

#### 文字色
| 变量 | 用途 |
|------|------|
| `--color-text-primary` | 主要文字 |
| `--color-text-secondary` | 次要文字 |
| `--color-text-muted` | 弱化文字（标签、提示） |
| `--color-text-inverse` | 反色文字（用于深色背景） |

#### 边框色
| 变量 | 用途 |
|------|------|
| `--color-border` | 默认边框 |
| `--color-border-light` | 浅色边框 |

#### 强调色
| 变量 | 用途 |
|------|------|
| `--color-accent` | 主强调色（链接、按钮） |
| `--color-accent-hover` | 强调色悬停态 |
| `--color-accent-light` | 强调色浅色版 |

#### 状态色
| 变量 | 用途 |
|------|------|
| `--color-success` | 成功状态 |
| `--color-warning` | 警告状态 |
| `--color-error` | 错误状态 |

#### 阴影
| 变量 | 用途 |
|------|------|
| `--shadow-sm` | 小阴影 |
| `--shadow-md` | 中阴影 |
| `--shadow-lg` | 大阴影 |

## 使用规范

### ✅ 正确用法

```css
.sidebar {
  background-color: var(--color-bg-sidebar);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-sm);
}

.title {
  color: var(--color-text-primary);
}

.button:hover {
  background-color: var(--color-bg-hover);
}
```

### ❌ 禁止用法

```css
/* 禁止硬编码颜色 */
.sidebar {
  background-color: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

/* 禁止使用 Tailwind 颜色类 */
<div class="bg-white text-gray-700">
```

### 例外情况

以下可以硬编码：
- **渐变色**：如头像背景 `linear-gradient(135deg, #60a5fa, #a78bfa)`
- **交通灯按钮**：使用专用变量 `--traffic-close` 等
- **品牌色**：固定不变的品牌标识色

## 添加新变量

1. 在 `themes/variables.css` 的 `:root` 中添加亮色值
2. 在 `.dark` 中添加对应暗色值
3. 更新本文档的变量列表

```css
:root {
  --color-new-variable: #value;
}

.dark {
  --color-new-variable: #dark-value;
}
```

## 组件样式规范

### 样式位置

- 所有样式写在 `<style scoped>` 中
- 使用语义化 class 名，不使用 Tailwind 工具类
- 通过 CSS 变量引用主题色

### class 命名

使用 **BEM 简化版**：

```css
/* 块 */
.sidebar { }

/* 元素 */
.sidebar-header { }
.sidebar-nav { }

/* 状态 */
.nav-item { }
.nav-item.active { }
.nav-item:hover { }
```

## 检查清单

添加新组件时确认：

- [ ] 所有颜色使用 CSS 变量
- [ ] 暗色模式下显示正常
- [ ] 悬停/激活状态使用对应变量
- [ ] 阴影使用 `--shadow-*` 变量
- [ ] 无硬编码颜色值（除例外情况）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhaowendao2005) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
