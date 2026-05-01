## grep-search-discipline

> Grep 搜索必须用精确模式，搜不到时先扩大范围再下结论


# Grep 搜索纪律

## 核心规则

**Grep 搜索"无结果"不等于"不存在"。搜不到时，必须换更宽泛的关键词重试，确认确实不存在后才能下结论。**

这个规则源于真实事故：用 `specsTab` 搜索 EE Admin 的 i18n 词条，搜不到就断定"词条缺失"——实际上词条以嵌套格式 `specs: { tab: "..." }` 存在，只是搜索模式不匹配。

## 搜索策略

### 1. 先从代码中提取实际使用的字符串

搜索某个东西是否存在之前，先确认你要搜的关键词是什么。**从使用方（调用处）提取实际字符串**，而不是自己猜测命名格式。

```
# 错误：猜测 i18n 键名格式然后去搜
Grep: specsTab|specsSaved|specsValidation

# 正确：先从代码中看到 t('settings.specs.tab')，再用实际键名搜
Grep: settings\.specs\.
```

### 2. 第一次搜不到时必须扩大范围

| 搜不到的情况 | 扩大方式 |
|---|---|
| 搜具体格式（如 `specsTab`） | 改为搜最短核心词（如 `specs`） |
| 搜组合关键词 | 拆成单个关键词分别搜 |
| 限定了文件类型/路径 | 去掉限定，全项目搜 |
| 用了正则 | 改为纯文本搜索 |

### 3. 搜索结论必须附带搜索模式

报告"某个东西不存在"时，必须说明用了什么搜索模式。让结论可验证：

```
# 错误
"EE Admin 的 i18n 词条缺失"

# 正确
"用 `settings\.specs\.` 在 ee/nodeskclaw-frontend/src/i18n/locales/ 搜索，
zh-CN.ts 第 302-314 行找到完整 specs 词条"
```

## 禁止

- **禁止一次搜索无结果就下结论** — 必须至少换一次更宽泛的模式重试
- **禁止假设命名格式** — 从源码实际调用处提取搜索词，不要用自己猜测的格式
- **禁止省略搜索依据** — "不存在"的结论必须附带搜索模式和搜索范围

---
> Source: [NoDeskAI/nodeskclaw](https://github.com/NoDeskAI/nodeskclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
