## marktwain

> 代码质量和复杂度限制规则


# 代码质量和复杂度限制规则

在 `features/refactor` 分支下，所有新建和修改的文件都必须遵循以下严格的质量标准：

## 文件大小限制
- **单个文件不得超过 700 行**
- 如果文件超过此限制，必须拆分为多个更小的模块

## 函数复杂度限制
- **单个函数不得超过 100 行**
- 超过限制的函数必须重构为多个更小的函数
- 优先使用纯函数和单一职责原则

## 循环嵌套限制
- **不得超过两重循环嵌套**（不包含在内）
- 允许的循环结构：
  ```javascript
  // ✅ 允许：单层循环
  for (let i = 0; i < items.length; i++) { }
  
  // ✅ 允许：两层循环
  for (let i = 0; i < rows.length; i++) {
    for (let j = 0; j < cols.length; j++) { }
  }
  
  // ❌ 禁止：三层及以上循环
  for (let i = 0; i < x; i++) {
    for (let j = 0; j < y; j++) {
      for (let k = 0; k < z; k++) { } // 不允许
    }
  }
  ```

## 模块职责限制
- **一个模块（文件）不得超过两个及以上的职责**
- 每个文件应该只负责一个主要功能
- 如果发现多个职责，必须拆分为独立的模块

## 重构指导原则

### 文件拆分策略
1. **按功能拆分**：将不同功能分离到不同文件
2. **按层级拆分**：将 UI 组件、业务逻辑、数据访问层分离
3. **按职责拆分**：将工具函数、类型定义、常量分离

### 函数重构策略
1. **提取方法**：将长函数拆分为多个小函数
2. **消除重复**：提取公共逻辑到独立函数
3. **单一职责**：每个函数只做一件事

### 循环优化策略
1. **使用数组方法**：优先使用 `map`、`filter`、`reduce` 等
2. **提前退出**：使用 `break` 或 `return` 减少循环次数
3. **算法优化**：考虑使用更高效的算法替代嵌套循环

## 代码审查检查点

在提交代码前，请确保：
- [ ] 文件行数 ≤ 700 行
- [ ] 所有函数 ≤ 100 行
- [ ] 循环嵌套 ≤ 2 层
- [ ] 模块职责 ≤ 2 个
- [ ] 代码可读性和可维护性良好

## 示例

### ✅ 良好的代码结构
```javascript
// user.service.js - 单一职责：用户管理
export class UserService {
  async getUser(id) { /* < 100 行 */ }
  async updateUser(id, data) { /* < 100 行 */ }
}

// user.validator.js - 单一职责：用户数据验证
export class UserValidator {
  validateEmail(email) { /* < 100 行 */ }
  validatePassword(password) { /* < 100 行 */ }
}
```

### ❌ 需要重构的代码
```javascript
// ❌ 文件过大，职责过多
export class UserManager {
  // 用户管理 + 验证 + 邮件发送 + 文件处理 = 4个职责
  async getUser() { /* 150 行 */ } // 超过 100 行
  async processUsers() {
    for (let i = 0; i < users.length; i++) {
      for (let j = 0; j < groups.length; j++) {
        for (let k = 0; k < permissions.length; k++) { // 3层循环
          // 处理逻辑
        }
      }
    }
  }
}
```

遵循这些规则将确保代码库的可维护性、可读性和可扩展性。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zillionare) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
