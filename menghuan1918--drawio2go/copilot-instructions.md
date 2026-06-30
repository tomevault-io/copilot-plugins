## drawio2go

> `app/components/constants/` 目录集中管理 React 组件库的配置常量、枚举值和默认参数。这些常量控制组件的行为边界、样式约束和交互限制，对整个组件系统的一致性和可维护性至关重要。

# 组件常量模块

## 模块概述

`app/components/constants/` 目录集中管理 React 组件库的配置常量、枚举值和默认参数。这些常量控制组件的行为边界、样式约束和交互限制，对整个组件系统的一致性和可维护性至关重要。

## 文件结构

```
app/components/constants/
└── sidebar.ts          # 侧边栏尺寸约束常量
```

## 常量分类

### 侧边栏尺寸约束

**文件**: `sidebar.ts`

| 常量名              | 值   | 类型   | 用途                | 影响范围                         |
| ------------------- | ---- | ------ | ------------------- | -------------------------------- |
| `SIDEBAR_MIN_WIDTH` | 300  | number | 侧边栏最小宽度 (px) | 侧边栏拖拽调整、响应式布局约束   |
| `SIDEBAR_MAX_WIDTH` | 3000 | number | 侧边栏最大宽度 (px) | 侧边栏拖拽调整、防止超大屏幕溢出 |

## 主要常量说明

### SIDEBAR_MIN_WIDTH (300px)

**定义**:

```typescript
export const SIDEBAR_MIN_WIDTH = 300;
```

**用途**: 侧边栏拖拽调整时的最小宽度边界

**使用位置**:

- `UnifiedSidebar.tsx` - 在 `calculateWidth()` 函数中通过 `Math.max()` 确保宽度不低于此值

**约束逻辑**:

```typescript
const clampedMax = Math.min(SIDEBAR_MAX_WIDTH, viewportWidth);
return Math.max(SIDEBAR_MIN_WIDTH, Math.min(rawWidth, clampedMax));
```

**影响范围**:

- 侧边栏响应式布局（聊天、版本、设置标签页）
- 拖拽调整宽度时的交互约束
- 小屏幕设备上的侧边栏布局

**修改建议**:

- 如需适配小屏幕，可酌情降低至 250-280px
- 不建议低于 250px（影响内容可读性）
- 考虑国际化时调整（日文、中文内容需更多宽度）

---

### SIDEBAR_MAX_WIDTH (3000px)

**定义**:

```typescript
export const SIDEBAR_MAX_WIDTH = 3000;
```

**用途**: 侧边栏拖拽调整时的最大宽度边界

**使用位置**:

- `UnifiedSidebar.tsx` - 在 `calculateWidth()` 函数中通过 `Math.min()` 确保宽度不超过此值

**约束逻辑**:

```typescript
const clampedMax = Math.min(SIDEBAR_MAX_WIDTH, viewportWidth);
```

**影响范围**:

- 超大屏幕（>3000px）下的侧边栏宽度上限
- 防止侧边栏占据过多视口空间
- 保留足够空间给编辑器区域

**修改建议**:

- 典型值范围: 2000-4000px
- 如启用 4K/5K 显示器支持，可调整至 4000px
- 需同时考虑 `viewportWidth` 限制（实际最大值为两者最小值）

---

## 使用示例

### 示例 1：侧边栏宽度约束

在 `UnifiedSidebar.tsx` 中的拖拽处理逻辑：

```typescript
import { SIDEBAR_MIN_WIDTH, SIDEBAR_MAX_WIDTH } from "./constants/sidebar";

const calculateWidth = (clientX: number) => {
  const viewportWidth = window.innerWidth;
  const rawWidth = viewportWidth - clientX;
  const clampedMax = Math.min(SIDEBAR_MAX_WIDTH, viewportWidth);
  // 宽度被约束在 [300px, min(3000px, viewportWidth)]
  return Math.max(SIDEBAR_MIN_WIDTH, Math.min(rawWidth, clampedMax));
};
```

### 示例 2：响应式适配

在新组件中引用常量进行宽度计算：

```typescript
import { SIDEBAR_MIN_WIDTH, SIDEBAR_MAX_WIDTH } from "@/app/components/constants/sidebar";

function AdaptiveLayout() {
  const defaultWidth = Math.min(400, SIDEBAR_MAX_WIDTH);
  const validatedWidth = Math.max(SIDEBAR_MIN_WIDTH, defaultWidth);

  return (
    <div style={{ width: `${validatedWidth}px` }}>
      {/* 响应式内容 */}
    </div>
  );
}
```

---

## 注意事项

### 1. 常量修改的全局影响

- **破坏性修改**: 修改任何常量都会影响所有引用该常量的组件
- **必须测试**: 修改后需在各屏幕尺寸（320px / 768px / 1024px / 1440px / 4K）测试
- **文档同步**: 修改常量时务必更新本文档和相关组件注释

### 2. 宽度约束的相互关系

```
设备宽度 (viewportWidth)
├── 约束上界: min(SIDEBAR_MAX_WIDTH, viewportWidth) = 3000px (典型)
└── 约束下界: SIDEBAR_MIN_WIDTH = 300px (固定)

计算逻辑: clamp(width, 300, min(3000, viewportWidth))
```

**关键点**:

- 实际最大宽度由设备宽度决定，不会超过 `viewportWidth`
- 即使 `SIDEBAR_MAX_WIDTH = 3000px`，在 1440px 的设备上实际上限也只有 1440px

### 3. 响应式设计考虑

修改常量前需考虑以下场景：

| 设备类型     | 宽度      | 适配建议                |
| ------------ | --------- | ----------------------- |
| 手机（竖屏） | 320-480px | `MIN=250px, MAX=400px`  |
| 平板         | 600-900px | `MIN=300px, MAX=600px`  |
| 笔记本       | 1440px    | `MIN=300px, MAX=1200px` |
| 超宽屏       | 3000px+   | 需调整 `MAX_WIDTH`      |

### 4. 国际化影响

不同语言的内容宽度需求差异：

- **英文**: 相对紧凑
- **中文/日文**: 需更多宽度（推荐 +20-30px）
- **阿拉伯文**: 需调整布局方向

### 5. 类型安全

所有常量均为基本类型（`number`），在引用时：

```typescript
// ✅ 安全用法
const width: number = SIDEBAR_MIN_WIDTH;
const maxWidth = Math.min(SIDEBAR_MAX_WIDTH, 2000);

// ❌ 避免
const width = SIDEBAR_MIN_WIDTH as any; // 类型断言会失效
```

### 6. 性能考虑

- 常量在模块加载时初始化，零运行时开销
- 引用常量代替硬编码值可提升代码可维护性
- 避免在 React 组件内多次计算相同的约束

---

## 相关文档

- **组件库文档**: `app/components/AGENTS.md` - UnifiedSidebar 组件详细说明
- **设计系统**: `app/styles/AGENTS.md` - UI 设计令牌和响应式断点
- **i18n 配置**: `app/i18n/AGENTS.md` - 国际化对宽度的影响

---

## 添加新常量的指南

若需新增常量：

1. **在现有文件中新增** (优先)

   ```typescript
   // sidebar.ts
   export const SIDEBAR_MIN_WIDTH = 300;
   export const SIDEBAR_MAX_WIDTH = 3000;
   export const SIDEBAR_DEFAULT_WIDTH = 400; // 新增
   ```

2. **创建新文件** (当分类明显不同时)

   ```typescript
   // app/components/constants/modal.ts
   export const MODAL_MIN_WIDTH = 320;
   export const MODAL_MAX_WIDTH = 900;
   ```

3. **更新本文档**
   - 在相应分类中添加表格行
   - 补充使用示例和注意事项

4. **添加类型注释**
   ```typescript
   /** 侧边栏最小宽度 (px) - 确保移动端可用性 */
   export const SIDEBAR_MIN_WIDTH = 300;
   ```

---

## 版本历史

- **2025-11-30**: 初版创建，文档化 sidebar.ts 常量

---
> Source: [Menghuan1918/drawio2go](https://github.com/Menghuan1918/drawio2go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
