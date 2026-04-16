## zz-ui-differ-plugins

> 前端质量体系-代码圈复杂度规则


# 前端代码圈复杂度质量规则

## 核心标准

### 单函数复杂度要求

- **硬性要求**: 单个函数的圈复杂度必须 **< 15**
- **推荐标准**: 单个函数的圈复杂度应 **< 10**

### 嵌套函数复杂度要求

嵌套复杂度比例 = (主函数复杂度 + 所有子函数复杂度) / (主函数代码行数 + 所有子函数代码行数)

- **硬性要求**: 嵌套复杂度比例必须 **< 25%**
- **推荐标准**: 嵌套复杂度比例应 **< 20%**

## 优化策略

### 1. 使用解构赋值和可选链

```javascript
// ❌ 避免深层嵌套判断
if (user && user.profile && user.profile.settings && user.profile.settings.theme) {
  // ...
}

// ✅ 使用解构和可选链
const { theme = 'default' } = user?.profile?.settings || {}
```

### 2. 使用查找表替代 if-else

```javascript
// ❌ 避免长 if-else 链
if (userType === 'vip') { /* ... */ }
else if (userType === 'regular') { /* ... */ }

// ✅ 使用策略模式
const handlers = { vip: handleVip, regular: handleRegular }
handlers[userType]?.()
```

### 3. 函数拆分

- 将复杂函数拆分为多个小函数
- 每个函数职责单一，复杂度 < 10
- 使用组合模式重构大函数

## 违规处理

- **警告**: 复杂度 10-14，嵌套复杂度 20%-24%
- **错误**: 复杂度 ≥15，嵌套复杂度 ≥25%
- 必须修复后才能提交代码

---

**注意**: 此规则为强制要求，所有新代码必须遵守。已有代码逐步重构，优先级按业务重要性和修改频率确定。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JacobZyy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
