## next-demo

> 项目使用严格的 TypeScript 配置（参考 [tsconfig.json](mdc:tsconfig.json)）：

# TypeScript 编码规范

## 基础配置标准

项目使用严格的 TypeScript 配置（参考 [tsconfig.json](mdc:tsconfig.json)）：

- 启用 `strict: true` 严格模式
- 使用 `@/*` 路径别名
- 目标版本：ES2017+

## 类型定义规范

### 1. 组件 Props 接口

```typescript
// ✅ 正确：明确的接口定义
interface ButtonProps
  extends Omit<
    React.ButtonHTMLAttributes<HTMLButtonElement>,
    | "onDrag"
    | "onDragEnd"
    | "onDragEnter"
    | "onDragExit"
    | "onDragLeave"
    | "onDragOver"
    | "onDragStart"
    | "onDrop"
    | "onAnimationStart"
    | "onAnimationEnd"
    | "onAnimationIteration"
  > {
  variant?: "primary" | "secondary" | "outline" | "ghost";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
  children: React.ReactNode;
}

// ❌ 避免：使用any类型
interface BadProps {
  data: any;
  onClick: any;
}
```

### 2. 函数类型注解

```typescript
// ✅ 正确：明确的函数签名
const Header: React.FC = () => {
  const [isMenuOpen, setIsMenuOpen] = useState<boolean>(false);

  const handleToggle = (open: boolean): void => {
    setIsMenuOpen(open);
  };

  return <div>...</div>;
};

// ✅ 正确：泛型函数
export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs));
}
```

### 3. 对象和数组类型

```typescript
// ✅ 正确：明确的数组类型
const navItems: Array<{ name: string; href: string }> = [
  { name: '首页', href: '/' },
  { name: '团队介绍', href: '/team' },
];

// ✅ 正确：使用接口定义复杂对象
interface NavItem {
  name: string;
  href: string;
  icon?: React.ComponentType;
}

const navItems: NavItem[] = [...];
```

## 组件开发规范

### 1. 函数组件定义

```typescript
// ✅ 推荐：使用函数组件 + TypeScript
const Button: React.FC<ButtonProps> = ({
  variant = "primary",
  size = "md",
  loading = false,
  disabled,
  children,
  className = "",
  ...props
}) => {
  // 组件逻辑
  return <button {...props}>{children}</button>;
};

export default Button;
```

### 2. 客户端组件标识

```typescript
// ✅ 需要使用浏览器API或状态的组件
"use client";

import React, { useState } from "react";

const InteractiveComponent: React.FC = () => {
  const [state, setState] = useState<string>("");
  // ...
};
```

### 3. 服务器组件（默认）

```typescript
// ✅ 服务器组件：不需要'use client'指令
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "页面标题",
  description: "页面描述",
};

export default function Page() {
  return <div>服务器渲染内容</div>;
}
```

## 错误处理和类型安全

### 1. 可选链和空值处理

```typescript
// ✅ 正确：使用可选链
const userName = user?.profile?.name ?? "未知用户";

// ✅ 正确：类型守卫
function isValidEmail(email: string): boolean {
  return email.includes("@") && email.length > 5;
}
```

### 2. 联合类型和类型守卫

```typescript
// ✅ 正确：联合类型定义
type Status = "loading" | "success" | "error";

interface ApiResponse<T> {
  status: Status;
  data?: T;
  error?: string;
}
```

## 命名约定

1. **变量和函数**：camelCase

   - `isMenuOpen`, `handleClick`, `fetchUserData`

2. **组件和类型**：PascalCase

   - `Button`, `Header`, `ButtonProps`

3. **常量**：UPPER_SNAKE_CASE

   - `API_BASE_URL`, `MAX_RETRY_COUNT`

4. **私有属性**：下划线前缀
   - `_private`, `_internalState`

## 导出规范

```typescript
// ✅ 推荐：默认导出主要组件
export default Button;

// ✅ 命名导出工具函数
export { cn, formatDate, validateEmail };

// ✅ 类型导出
export type { ButtonProps, NavItem };
```

## 性能优化

1. **使用 React.memo 进行组件优化**
2. **合理使用 useState 和 useEffect 类型**
3. **避免在组件内定义复杂对象**

遵循这些 TypeScript 规范可以确保代码的类型安全性、可维护性和团队协作效率。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Daotin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
