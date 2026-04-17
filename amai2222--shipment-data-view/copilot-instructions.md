## shipment-data-view

> **AI助手必须使用简体中文进行所有交流和输出**

# Cursor AI 助手规则配置

## 🌐 语言设置（最重要）

**AI助手必须使用简体中文进行所有交流和输出**

- ✅ AI助手的对话、解释、建议全部使用简体中文
- ✅ 代码注释使用简体中文
- ✅ Git提交信息使用简体中文
- ✅ 文档和README使用简体中文
- ✅ 错误提示和日志信息使用简体中文
- ✅ 函数说明和注释使用简体中文

## 💬 AI助手交流规范

### 对话语言
- **必须使用简体中文**回答用户问题
- **必须使用简体中文**解释代码逻辑
- **必须使用简体中文**提供建议和方案
- 技术术语可保留英文（如React、TypeScript、SQL）
- 代码变量名使用英文，但注释用中文

### 示例
```typescript
// ✅ 正确：中文注释
// 获取用户列表并按创建时间排序
const getUserList = async () => {
  // 调用API获取数据
  const response = await fetch('/api/users');
  return response.json();
};

// ❌ 错误：英文注释
// Get user list and sort by created time
const getUserList = async () => {
  // Call API to fetch data
  const response = await fetch('/api/users');
  return response.json();
};
```

## 📝 Git提交信息规范

### 格式要求
- **必须使用简体中文**
- 格式：`动词 + 具体内容`
- 简洁明了，一句话说明改动

### 提交信息示例

#### ✅ 好的提交信息（中文）
```
添加开票流程状态分区功能
修复付款审核页面筛选器默认值
优化申请单列表的分割线样式
更新StatusBadge组件状态配置
重构开票审批函数同步更新运单状态
```

#### ❌ 不好的提交信息（英文）
```
Add invoice workflow status partition feature
Fix payment audit page filter default value
Optimize application list divider style
```

## 🎨 代码规范

### 命名规范
- **变量名**：使用英文驼峰命名（camelCase）
- **函数名**：使用英文驼峰命名（camelCase）
- **组件名**：使用英文帕斯卡命名（PascalCase）
- **注释**：使用简体中文
- **UI文本**：使用简体中文

### 示例
```typescript
// ✅ 正确示例
interface PaymentRequest {
  id: string;           // 申请单ID
  status: string;       // 申请单状态
  amount: number;       // 申请金额
  createdAt: string;    // 创建时间
}

// 处理付款审批
const handleApproval = async (requestId: string) => {
  // 检查权限
  if (!hasPermission) {
    toast({ title: "权限不足", description: "您没有审批权限" });
    return;
  }
  
  // 调用审批接口
  const result = await approvePayment(requestId);
  
  // 显示成功提示
  toast({ title: "审批成功", description: "付款申请已通过审批" });
};
```

## 📄 文档规范

### 文档语言
- **必须使用简体中文**编写所有文档
- Markdown标题、正文、列表全部使用中文
- 代码示例中的注释使用中文
- 技术术语可保留英文（如API、SQL、RPC）

### 文档结构
```markdown
# 功能名称（中文）

## 功能概述
用简体中文描述功能...

## 使用方法
1. 第一步操作说明（中文）
2. 第二步操作说明（中文）

## 代码示例
\`\`\`typescript
// 中文注释
const example = () => {
  // 具体实现
};
\`\`\`
```

## 🔧 SQL函数注释

### 函数说明
- 函数注释使用简体中文
- 参数说明使用简体中文
- 返回值说明使用简体中文

### 示例
```sql
-- ============================================================================
-- 函数名称：审批付款申请
-- 功能描述：审批付款申请并同步更新运单状态
-- 参数：
--   p_request_id: 申请单ID
-- 返回：JSON对象包含操作结果
-- ============================================================================
CREATE OR REPLACE FUNCTION approve_payment_request(
    p_request_id TEXT
)
RETURNS JSONB
AS $$
BEGIN
    -- 检查权限
    IF NOT is_finance_or_admin() THEN
        RAISE EXCEPTION '权限不足：只有财务或管理员可以审批';
    END IF;
    
    -- 更新申请单状态
    UPDATE payment_requests 
    SET status = 'Approved'
    WHERE request_id = p_request_id;
    
    RETURN jsonb_build_object('success', true, 'message', '审批成功');
END;
$$;

COMMENT ON FUNCTION approve_payment_request IS '审批付款申请并同步更新运单状态';
```

## 🎯 用户界面文本

### UI文本要求
- **所有按钮文字**：简体中文
- **所有提示信息**：简体中文
- **所有表单标签**：简体中文
- **所有错误信息**：简体中文

### 示例
```typescript
// ✅ 正确
<Button>提交申请</Button>
toast({ title: "操作成功", description: "数据已保存" });
<Label>申请单号</Label>

// ❌ 错误
<Button>Submit</Button>
toast({ title: "Success", description: "Data saved" });
<Label>Request ID</Label>
```

## 📊 数据库对象命名

### 表名和字段名
- 使用英文下划线命名（snake_case）
- 注释使用简体中文

```sql
-- ✅ 正确
CREATE TABLE payment_requests (
    id UUID PRIMARY KEY,
    request_id TEXT NOT NULL,      -- 申请单编号
    status TEXT NOT NULL,          -- 申请单状态
    created_at TIMESTAMP,          -- 创建时间
    total_amount DECIMAL            -- 申请金额
);

COMMENT ON TABLE payment_requests IS '付款申请单表';
COMMENT ON COLUMN payment_requests.status IS '申请单状态：Pending-待审核, Approved-已审批待支付, Paid-已支付';
```

## 🚀 AI助手工作流程

当用户提问时，AI助手应该：

1. **使用中文理解**用户需求
2. **使用中文解释**解决方案
3. **使用中文注释**生成代码
4. **使用中文编写**提交信息
5. **使用中文创建**文档

## 💡 特殊情况处理

### 技术术语保留英文
- React, TypeScript, JavaScript
- SQL, RPC, API, REST
- Git, GitHub, Supabase
- UUID, JSON, JSONB

### 混合使用示例
```
// ✅ 正确：术语用英文，说明用中文
// 调用Supabase RPC函数获取付款申请列表
const { data, error } = await supabase.rpc('get_payment_requests');

// 使用React Hook管理状态
const [requests, setRequests] = useState<PaymentRequest[]>([]);
```

---

## ✅ 配置验证

当你看到AI助手：
- ✅ 用中文回答问题
- ✅ 用中文解释代码
- ✅ 生成中文注释
- ✅ 生成中文提交信息
- ✅ 创建中文文档

**说明配置生效！** 🎉

---

## 📅 日期同步规则（重要）

### 自动同步当前日期

**AI助手必须使用用户当前的系统日期，而不是AI训练数据的日期。**

#### 日期配置文件
- **配置文件**：项目根目录 `.cursor-date`
- **格式**：`CURRENT_DATE=YYYY-MM-DD`
- **示例**：`CURRENT_DATE=2025-11-16`
- **自动更新**：运行 `npm run update-date` 自动更新为系统当前日期

#### 日期使用规则

1. **每次对话开始时**：
   - AI助手必须首先读取 `.cursor-date` 文件
   - 使用文件中的 `CURRENT_DATE` 作为当前日期
   - 如果文件不存在或日期格式错误，AI助手应该：
     1. 首先尝试运行 `npm run update-date` 自动更新日期
     2. 如果自动更新失败，询问用户当前日期

2. **创建新文件时**：
   - SQL迁移文件：使用 `YYYYMMDD_` 前缀（如：`20251116_`）
   - 文档中的日期：使用 `YYYY-MM-DD` 格式（如：`2025-11-16`）
   - 代码注释中的日期：使用 `YYYY-MM-DD` 格式

3. **更新日期时**：
   - 当用户提供新日期时，AI助手应该：
     1. 更新 `.cursor-date` 文件
     2. 更新所有相关文档和代码中的日期
     3. 确认日期已同步

4. **日期格式**：
   - 标准格式：`YYYY-MM-DD`（如：`2025-11-16`）
   - 文件名格式：`YYYYMMDD`（如：`20251116`）
   - 文档格式：`YYYY-MM-DD`（如：`2025-11-16`）

#### 示例

```markdown
# 当创建新迁移文件时
文件名：supabase/migrations/20251116_add_new_feature.sql
注释：-- 日期：2025-11-16

# 当创建新文档时
文档头部：
## 📅 创建日期
2025-11-16

**最后更新**：2025-11-16
```

#### 注意事项

- ✅ **必须**使用 `.cursor-date` 文件中的日期
- ✅ **必须**在创建新文件时使用当前日期
- ✅ **必须**在更新文档时同步日期
- ❌ **禁止**使用AI训练数据中的日期
- ❌ **禁止**使用硬编码的旧日期

---

**配置文件位置**：项目根目录 `.cursorrules`  
**日期配置文件**：项目根目录 `.cursor-date`  
**配置时间**：2025-11-16  
**状态**：✅ 已生效

🌟 **现在Cursor AI助手将全部使用简体中文输出，并自动同步您的当前日期！**

---

## 🛡️ 代码质量与类型安全（最高优先级）

**AI 在生成代码时必须强制执行以下检查，违反以下规则的代码将被视为无效：**

### 1. TypeScript 严格模式

- ❌ **严禁使用 `any`**：遇到类型困难时，必须定义 `interface` 或 `type`。如果必须使用临时类型，请使用 `unknown` 并配合类型守卫。

  ```typescript
  // ❌ 错误：使用 any
  catch (error: any) {
    console.error(error.message);
  }

  // ✅ 正确：使用 unknown 和类型守卫
  catch (error: unknown) {
    const errorMessage = error instanceof Error ? error.message : '未知错误';
    console.error(errorMessage);
  }
  ```

- ✅ **Props 类型定义**：所有 React 组件必须定义 Props 接口。例如 `interface ButtonProps { ... }`。

  ```typescript
  // ✅ 正确：定义 Props 接口
  interface ButtonProps {
    label: string;
    onClick: () => void;
    disabled?: boolean;
  }

  export function Button({ label, onClick, disabled }: ButtonProps) {
    return <button onClick={onClick} disabled={disabled}>{label}</button>;
  }
  ```

- ✅ **事件类型**：必须使用正确的 React 事件类型，例如 `React.ChangeEvent<HTMLInputElement>`，而不是 `any`。

  ```typescript
  // ❌ 错误：使用 any
  const handleChange = (e: any) => {
    setValue(e.target.value);
  };

  // ✅ 正确：使用正确的 React 事件类型
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };
  ```

### 2. 导入/导出与路径修复

- ✅ **自动检查路径**：在生成 `import` 语句前，必须检查文件是否存在。优先使用 `@/` 别名路径。

  ```typescript
  // ✅ 正确：使用 @/ 别名
  import { Button } from '@/components/ui/button';
  import { useToast } from '@/hooks/use-toast';

  // ❌ 错误：使用相对路径（除非必要）
  import { Button } from '../../../components/ui/button';
  ```

- ✅ **修正导入方式**：根据 `package.json` 检查第三方库是 `default` 导出还是 `named` 导出。

  ```typescript
  // ❌ 错误：default 导入（如果是命名导出）
  import Button from '@/components/ui/button';

  // ✅ 正确：named 导入
  import { Button } from '@/components/ui/button';
  ```

### 3. 构建与语法错误预防

- ✅ **JSX 闭合检查**：输出前自检所有 JSX 标签是否正确闭合。

  ```typescript
  // ❌ 错误：标签未闭合
  <div>
    <Button>提交</Button>
  // 缺少 </div>

  // ✅ 正确：所有标签正确闭合
  <div>
    <Button>提交</Button>
  </div>
  ```

- ✅ **Hooks 规则**：确保 `useEffect` 依赖项完整，不要在循环或条件语句中调用 Hooks。

  ```typescript
  // ❌ 错误：缺少依赖项
  useEffect(() => {
    loadData();
  }, []); // 如果 loadData 在组件内定义，应该包含在依赖中

  // ✅ 正确：包含所有依赖项，或使用 eslint-disable 注释说明原因
  useEffect(() => {
    loadData();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []); // loadData 是稳定的函数，不需要添加到依赖
  ```

- ✅ **React 18 兼容性**：使用 `createRoot` 而不是 `ReactDOM.render`，确保组件返回值类型为 `JSX.Element` 或 `React.ReactNode`。

  ```typescript
  // ✅ 正确：组件返回类型
  export function MyComponent(): JSX.Element {
    return <div>内容</div>;
  }

  // ✅ 正确：使用 React.ReactNode
  export function MyComponent(): React.ReactNode {
    return <div>内容</div>;
  }
  ```

### 4. 自我修正机制

如果生成的代码可能导致构建失败（例如引用了不存在的变量或属性），AI 必须：

1. **停止生成**：立即停止当前代码生成。
2. **重新分析上下文**：检查相关类型定义、接口、导入路径等。
3. **重新生成修复后的代码**：生成修复后的代码，并附加说明："已修复潜在的类型错误..."。

#### 修正流程示例

```typescript
// 第一步：发现问题
// 错误：record.driverPhone 不存在，应该是 driver_phone

// 第二步：检查类型定义
// 查看 LogisticsRecord 接口，确认字段名为 driver_phone

// 第三步：修复并说明
// 已修复潜在的类型错误：将 record.driverPhone 改为 record.driver_phone
// 以匹配 LogisticsRecord 接口的 snake_case 命名规范
```

### 5. 类型安全最佳实践

- ✅ **接口优先**：优先使用 `interface` 定义类型，而不是 `type`（除非需要联合类型或工具类型）。

  ```typescript
  // ✅ 正确：使用 interface
  interface User {
    id: string;
    name: string;
    email: string;
  }

  // ✅ 正确：联合类型使用 type
  type Status = 'pending' | 'approved' | 'rejected';
  ```

- ✅ **类型推断**：充分利用 TypeScript 的类型推断，避免不必要的类型注解。

  ```typescript
  // ✅ 正确：利用类型推断
  const users: User[] = [];
  const count = users.length; // count 自动推断为 number

  // ❌ 错误：不必要的类型注解
  const count: number = users.length;
  ```

- ✅ **类型守卫**：使用类型守卫确保类型安全。

  ```typescript
  // ✅ 正确：使用类型守卫
  function isError(value: unknown): value is Error {
    return value instanceof Error;
  }

  try {
    // ...
  } catch (error: unknown) {
    if (isError(error)) {
      console.error(error.message);
    }
  }
  ```

---

## ⚠️ 违反规则的后果

如果生成的代码违反以上任何规则：

1. **代码将被视为无效**
2. **必须立即修正**
3. **修正后必须说明修正内容**

---

## 🚫 严格禁止项（零容忍）

以下行为被严格禁止，违反将导致代码被视为无效：

### 1. 禁止使用 `any` 类型

- ❌ **绝对禁止**：使用 `any` 类型来消除编译错误
- ✅ **必须**：明确定义接口或类型。如果类型复杂，必须构建它
- ✅ **替代方案**：使用 `unknown` 配合类型守卫

```typescript
// ❌ 严格禁止
const handleError = (error: any) => {
  console.error(error.message);
};

// ✅ 正确做法
const handleError = (error: unknown) => {
  if (error instanceof Error) {
    console.error(error.message);
  } else {
    console.error('未知错误');
  }
};
```

### 2. 禁止语法错误

- ✅ **必须**：仔细检查所有括号 `()`、花括号 `{}` 和方括号 `[]`
- ✅ **必须**：确保所有 JSX 标签正确闭合
- ✅ **必须**：在输出代码前进行语法自检

```typescript
// ❌ 语法错误：标签未闭合
<div>
  <Button>提交</Button>

// ✅ 正确：所有标签正确闭合
<div>
  <Button>提交</Button>
</div>
```

### 3. 禁止导入错误

- ✅ **必须验证**：导入的模块确实存在
- ✅ **必须检查**：库使用的是命名导出 `{ Component }` 还是默认导出 `Component`
- ❌ **禁止**：从 `dist` 或内部路径导入（除非必要）
- ✅ **必须**：根据项目结构正确使用相对路径（`./`, `../`）或别名路径（`@/`）

```typescript
// ❌ 错误：导入不存在的模块
import { NonExistent } from '@/components/non-existent';

// ❌ 错误：错误的导出方式
import Button from '@/components/ui/button'; // 如果是命名导出

// ✅ 正确：验证存在并使用正确的导出方式
import { Button } from '@/components/ui/button';
```

### 4. 禁止未使用的变量

- ✅ **必须**：立即清理定义但未使用的变量
- ✅ **必须**：移除未使用的导入

```typescript
// ❌ 错误：定义了但未使用
const unusedVariable = 'test';
const [unusedState, setUnusedState] = useState('');

// ✅ 正确：移除未使用的变量
// 只保留实际使用的变量
```

## 🛠️ 自动修复与验证策略

在输出任何代码之前，必须内部运行"虚拟 lint"检查：

### 1. TypeScript 检查

- ✅ **必须验证**：代码是否满足严格类型检查？
- ✅ **必须验证**：所有 Props 是否已定义？
- ✅ **必须验证**：所有类型是否匹配？

```typescript
// 内部检查流程：
// 1. 检查所有变量是否有类型注解或类型推断
// 2. 检查所有函数参数和返回值类型
// 3. 检查所有 Props 接口是否定义
// 4. 检查是否有类型不匹配
```

### 2. 构建检查

- ✅ **必须验证**：代码是否会破坏构建？
- ✅ **必须验证**：是否使用了类型上不存在的属性？
- ✅ **必须验证**：是否引用了不存在的变量或函数？

```typescript
// 内部检查流程：
// 1. 检查所有属性访问是否在类型定义中存在
// 2. 检查所有函数调用是否匹配函数签名
// 3. 检查所有变量引用是否已定义
```

### 3. 导入检查

- ✅ **必须验证**：路径是否正确（相对路径 `./`, `../` 或别名路径 `@/`）？
- ✅ **必须验证**：导入路径是否基于项目结构正确使用？

```typescript
// 内部检查流程：
// 1. 检查导入路径是否存在
// 2. 检查是否应该使用别名路径（@/）
// 3. 检查导出方式是否正确（named vs default）
```

## 📦 React 与生态系统最佳实践

### 文件扩展名

- ✅ **组件文件**：使用 `tsx` 扩展名
- ✅ **工具文件**：使用 `ts` 扩展名

### 类型导入

- ✅ **类型导入**：使用 `import type` 进行仅类型导入

```typescript
// ✅ 正确：使用 import type 导入类型
import type { User } from '@/types';
import { Button } from '@/components/ui/button';

// ❌ 错误：混合导入类型和值
import { User, getUser } from '@/types'; // 如果 User 只是类型
```

### React Hooks 导入

- ✅ **必须**：确保 Hooks（useState, useEffect）从 'react' 导入

```typescript
// ✅ 正确
import { useState, useEffect } from 'react';

// ❌ 错误
import useState from 'react'; // 错误：useState 不是默认导出
```

### 第三方库导入

- ✅ **必须**：根据文档正确导入组件
- ✅ **必须验证**：库的导入方式（如 `lucide-react`, `recharts`, `shadcn/ui`）

```typescript
// ✅ 正确：根据文档导入
import { Button } from '@/components/ui/button'; // shadcn/ui
import { Search } from 'lucide-react'; // lucide-react
import { LineChart } from 'recharts'; // recharts

// ❌ 错误：错误的导入方式
import Button from 'lucide-react'; // 错误：lucide-react 使用命名导出
```

## 🛡️ 错误恢复机制

如果用户粘贴了错误消息，必须：

### 1. 分析错误代码

- ✅ **必须**：分析具体的错误代码（如 TS2322, TS2339, TS2551）
- ✅ **必须**：理解错误的根本原因

```typescript
// 错误示例：TS2322: Type 'string' is not assignable to type 'number'
// 分析：类型不匹配，需要转换类型或修正类型定义
```

### 2. 不抑制错误

- ❌ **禁止**：使用 `@ts-ignore` 或 `@ts-expect-error` 来抑制错误
- ✅ **必须**：修复根本原因

```typescript
// ❌ 严格禁止
// @ts-ignore
const value: number = 'string';

// ✅ 正确：修复根本原因
const value: number = parseInt('string', 10);
// 或
const value: string = 'string';
```

### 3. 修复根本原因

- ✅ **必须**：修复类型不匹配
- ✅ **必须**：添加缺失的属性
- ✅ **必须**：修正导入路径

```typescript
// 错误：TS2339: Property 'name' does not exist on type 'User'
// 修复步骤：
// 1. 检查 User 接口定义
// 2. 添加缺失的 name 属性
// 3. 或修正属性访问（如 user.userName 而不是 user.name）
```

---

**最后更新**：2025-11-23  
**优先级**：最高（代码质量与类型安全）  
**执行级别**：零容忍（违反规则的代码将被视为无效）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amai2222) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
