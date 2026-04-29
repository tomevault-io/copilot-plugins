## create-react-web

> - 始终声明变量和函数的类型（参数和返回值）


# TypeScript 规范

## 基本原则
- 始终声明变量和函数的类型（参数和返回值）
- 避免使用 `any` 或 `unknown`，除非绝对必要
- 创建必要的类型定义，充分利用类型推导
- 优先使用 interface 而不是 type
- 避免使用枚举（enum），使用字面量类型或常量对象
- 避免使用类型断言（`as` 或 `!`）

## 类型定义

### Interface vs Type
```typescript
// ✅ 推荐：使用 interface
interface User {
  id: string;
  name: string;
  email: string;
}

// ✅ 推荐：联合类型使用 type
type UserRole = 'admin' | 'user' | 'guest';
type ApiResponse<T> = { data: T; code: number };

// ❌ 避免：使用 enum
enum UserRole {
  Admin = 'ADMIN',
  User = 'USER',
}
```

### 泛型使用
```typescript
// ✅ 推荐：清晰的泛型参数
function getData<T>(id: string): Promise<T> {
  return fetch(`/api/data/${id}`).then(res => res.json());
}

// 常用泛型参数命名
T       // Type - 通用类型
K, V    // Key, Value - 键值对
E       // Element - 元素类型
P       // Props - 组件属性
S       // State - 状态类型
```

### 函数类型
```typescript
// ✅ 推荐：完整的类型注解
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✅ 推荐：箭头函数类型
const formatDate = (date: Date): string => {
  return date.toISOString();
};

// ✅ 推荐：异步函数返回 Promise
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

## 函数设计

### 函数长度和复杂度
- 单个函数少于 100 行
- 避免嵌套代码块，使用提前返回
- 使用高阶函数（map, filter, reduce）避免函数嵌套

```typescript
// ❌ 不好：深层嵌套
function processUser(user: User) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        // 处理逻辑
      }
    }
  }
}

// ✅ 好：提前返回
function processUser(user: User) {
  if (!user) return;
  if (!user.isActive) return;
  if (!user.hasPermission) return;
  
  // 处理逻辑
}
```

### 参数设计
```typescript
// ❌ 不好：参数过多
function createUser(
  name: string,
  email: string,
  age: number,
  address: string
) { }

// ✅ 好：使用对象参数（RO-RO 模式）
interface CreateUserParams {
  name: string;
  email: string;
  age: number;
  address: string;
}

function createUser(params: CreateUserParams) { }

// ✅ 好：使用默认参数
function fetchData(url: string, timeout = 5000) { }
```

## 错误处理

```typescript
// ✅ 推荐：完整的错误处理
async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch user: ${response.statusText}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof Error) {
      console.error('Fetch user error:', error.message);
    }
    throw error;
  }
}

// ✅ 推荐：处理边界情况
function getFirstItem<T>(arr: T[]): T | undefined {
  if (arr.length === 0) {
    return undefined;
  }
  return arr[0];
}
```

## 类型安全

### 避免 any
```typescript
// ❌ 不好：使用 any
function process(data: any) {
  return data;
}

// ✅ 好：使用泛型
function process<T>(data: T): T {
  return data;
}

// ✅ 好：使用 unknown（如果类型真的未知）
function parse(json: string): unknown {
  return JSON.parse(json);
}
```

### 类型守卫
```typescript
// ✅ 推荐：使用类型守卫
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj
  );
}

// 使用
if (isUser(data)) {
  console.log(data.name); // TypeScript 知道这是 User 类型
}
```

## 强制行为 ✅
1. 必须为所有变量和函数声明类型
2. 必须避免使用 `any`（除非充分理由）
3. 必须处理错误和边界情况
4. 必须使用对象参数（超过 3 个参数时）
5. 必须保持函数简短（< 100 行）

## 禁止行为 ❌
1. 禁止忽略类型错误
2. 禁止使用 `@ts-ignore`（除非充分理由）
3. 禁止滥用类型断言（`as`）
4. 禁止创建超过 100 行的函数
5. 禁止使用 enum（使用字面量类型）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YorenLee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
