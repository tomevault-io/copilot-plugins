## no-markdown-generation

> 不要主动生成 markdown 文件


# Markdown 文件生成规则

除非用户明确要求，否则不要生成 markdown 文件。

## 规则说明

- ❌ **禁止**：主动创建 `.md` 文件（如 README.md、文档文件等）
- ✅ **允许**：用户明确请求时才创建 markdown 文件
- ✅ **允许**：编辑已存在的 markdown 文件

## 示例

### ❌ 错误做法

```
用户：创建一个新的 React 组件
AI：我会创建组件并添加 README.md 说明文档...  ← 不要这样做
```

### ✅ 正确做法

```
用户：创建一个新的 React 组件
AI：我会创建组件文件...  ← 只创建必要的代码文件
```

### ✅ 例外情况

```
用户：请创建一个 README.md 文件说明这个项目
AI：好的，我会创建 README.md...  ← 用户明确要求时可以创建
```

---
> Source: [mybricks/mybricks-ide-ui-plugin-vscode](https://github.com/mybricks/mybricks-ide-ui-plugin-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
