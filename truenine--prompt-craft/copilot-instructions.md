## prompt-craft

> TypeScript规范


## 核心规范
- 禁用`any`类型，必须显式声明类型
- 避免三层以上嵌套，重构为函数
- 使用`void 0`替代`undefined`
- 禁用非空断言

## 性能准则
- 循环内禁止创建函数/对象
- 用`forEach`替代`for...of`
- 热路径禁用`try...catch`

## 最佳实践
- 用可选链`?.`替代`&&`链
- 用空值合并`??`替代`||`判断

## 模块规范
- 使用ES模块，禁用CommonJS
- 禁用`require`语句
- 禁用`import * as`导入

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TrueNine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
