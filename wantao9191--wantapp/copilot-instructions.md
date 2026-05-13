## coding-standards

> TypeScript 和 React 编码规范


# 编码规范

## TypeScript 规范

- 使用严格的 TypeScript 配置 (`strict: true`)
- 优先使用 `interface` 定义对象类型
- 使用 `type` 定义联合类型和复杂类型
- 所有函数参数和返回值都要有明确的类型注解
- 使用 `as const` 进行常量断言

## React 组件规范

- 使用函数式组件和 Hooks
- 组件名使用 PascalCase
- Props 接口以组件名 + Props 命名
- 使用 `React.FC` 或直接函数声明
- 优先使用 `useState` 和 `useEffect`

## 导入规范

- 使用绝对路径导入 (`@/` 别名)
- 第三方库导入在前，本地导入在后
- 按字母顺序排列导入
- 使用命名导入而非默认导入（除非必要）

## 文件命名

- 组件文件使用 PascalCase: `UserProfile.tsx`
- Hook 文件使用 camelCase: `useAuth.ts`
- 工具文件使用 kebab-case: `auth-helper.ts`
- 类型文件使用 kebab-case: `database.ts`

## 代码风格

- 使用分号结尾
- 使用 2 个空格缩进
- 字符串优先使用单引号
- 对象和数组末尾添加逗号
- 使用 Prettier 进行代码格式化

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
