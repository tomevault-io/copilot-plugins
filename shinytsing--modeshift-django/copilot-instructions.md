## javascript-typescript

> JavaScript和TypeScript语言特定规则


# JavaScript和TypeScript语言特定规则

基于awesome-cursorrules的JavaScript/TypeScript最佳实践。

## TypeScript最佳实践

### 类型注解
```typescript
// 函数类型注解
function processData(data: string[]): Promise<ProcessedData> {
  // 实现
}

// 接口定义
interface User {
  id: number;
  name: string;
  email: string;
  isActive?: boolean;
}

// 泛型使用
function createApiResponse<T>(data: T, success: boolean): ApiResponse<T> {
  return { data, success, timestamp: Date.now() };
}
```

### React组件规范
```typescript
import React, { FC, useState, useEffect } from 'react';

interface ComponentProps {
  title: string;
  onAction: (value: string) => void;
  children?: React.ReactNode;
}

const MyComponent: FC<ComponentProps> = ({ title, onAction, children }) => {
  const [state, setState] = useState<string>('');
  
  useEffect(() => {
    // 副作用处理
  }, []);
  
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  );
};

export default MyComponent;
```

## 代码组织

### 目录结构
```
src/
  components/
    ui/
      Button.tsx
      Input.tsx
    forms/
      LoginForm.tsx
  hooks/
    useApi.ts
    useLocalStorage.ts
  services/
    api.ts
    auth.ts
  types/
    index.ts
    api.ts
  utils/
    helpers.ts
    constants.ts
  pages/
    Home.tsx
    Profile.tsx
```

### 导入/导出规范
```typescript
// 优先使用命名导出
export const utilityFunction = () => {};
export const CONSTANT_VALUE = 'value';

// 默认导出用于主要组件
export default MyComponent;

// 导入顺序：第三方库 -> 内部模块 -> 相对路径
import React from 'react';
import { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { apiService } from '@/services/api';
import './styles.css';
```

## 错误处理

```typescript
// 异步函数错误处理
async function fetchUserData(id: string): Promise<User | null> {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    console.error('获取用户数据失败:', error);
    return null;
  }
}

// 类型安全的错误处理
type ApiError = {
  message: string;
  code: number;
};

function handleApiError(error: unknown): ApiError {
  if (error instanceof Error) {
    return { message: error.message, code: 500 };
  }
  return { message: '未知错误', code: 500 };
}
```

## 性能优化

### React性能优化
```typescript
// 使用React.memo优化重渲染
const ExpensiveComponent = React.memo<Props>(({ data }) => {
  return <div>{/* 渲染逻辑 */}</div>;
});

// 使用useMemo缓存计算结果
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

// 使用useCallback缓存函数
const handleClick = useCallback((id: string) => {
  onItemClick(id);
}, [onItemClick]);
```

### 代码分割
```typescript
// 懒加载组件
const LazyComponent = React.lazy(() => import('./LazyComponent'));

// 使用Suspense包装
<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

## 测试规范

```typescript
// 使用Jest和React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  it('应该正确渲染标题', () => {
    render(<MyComponent title="测试标题" onAction={jest.fn()} />);
    expect(screen.getByText('测试标题')).toBeInTheDocument();
  });
  
  it('应该处理点击事件', () => {
    const mockOnAction = jest.fn();
    render(<MyComponent title="测试" onAction={mockOnAction} />);
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockOnAction).toHaveBeenCalledWith('expected-value');
  });
});
```

## 代码质量

### ESLint配置
```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "react-hooks/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

### 命名规范
- 组件：PascalCase (`MyComponent`)
- 函数/变量：camelCase (`handleClick`)
- 常量：UPPER_SNAKE_CASE (`API_BASE_URL`)
- 接口：PascalCase，前缀I (`IUserData`)
- 类型：PascalCase (`UserData`)

## 现代JavaScript特性

```typescript
// 使用可选链和空值合并
const userName = user?.profile?.name ?? '未知用户';

// 使用解构赋值
const { id, name, email } = user;

// 使用模板字符串
const message = `欢迎 ${name}，您的ID是 ${id}`;

// 使用箭头函数
const users = data.map(user => ({
  ...user,
  displayName: `${user.firstName} ${user.lastName}`
}));
```

## 异步处理

```typescript
// 使用async/await
async function loadData() {
  try {
    const [users, posts] = await Promise.all([
      fetchUsers(),
      fetchPosts()
    ]);
    return { users, posts };
  } catch (error) {
    console.error('加载数据失败:', error);
    throw error;
  }
}

// 使用Promise.allSettled处理多个异步操作
const results = await Promise.allSettled([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);

const successfulResults = results
  .filter(result => result.status === 'fulfilled')
  .map(result => result.value);
```

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
