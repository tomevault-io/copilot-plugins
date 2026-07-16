## ctofconverter

> - 新页面URL统一不带尾部斜杠（如 `/guides/weather-temperature-guide`）

## 页面规范
- 响应式设计，移动端优先
- 新页面URL统一不带尾部斜杠（如 `/guides/weather-temperature-guide`）
- 旧页面保持现有URL不动

## 做新页面之前
1. 先做关键词调研 — 在谷歌 SERP 确认有真实搜索量、且竞争可打
2. 检查站内是否已有覆盖此内容的页面 — 避免重复
3. 确认该页面的「信息增益」— 用户在这页能看到其他站看不到什么？

## 内容创作流程
1. 确定关键词和搜索意图
2. 分析 SERP 前 5 名，看竞品做到什么程度
3. 规划信息增益点：你比前 5 名多提供什么价值？
4. 写 Answer Capsule（40-60字直接答案）
5. 写正文（H2/H3 分段，每段以 Answer Capsule 开头）
6. 添加结构化数据（表格/列表/Schema）
7. 写 FAQ（3-5 个真实问题，每个 40-60 字）
8. 添加内部链接

## 内容原则
- 零程序化生成：每页手动写，不用模板
- 零同义替换：不用 `textSpinner.ts`，不搞伪原创
- 以用户为中心，回答真实问题
- 内容真实、准确，标注数据来源
- 每页必须有信息增益（用户在这页能看到其他站看不到的价值）
- 不要写废话（如 "Temperature is a measure of how hot or cold something is"）

## GEO（面向 AI 搜索优化）
每个页面必须包含：
1. **Answer Capsule** — 每段 H2/H3 下放 40-60 字的直接答案段，方便 AI 提取引用
2. **结构化数据** — 表格/列表/FAQ Schema/HowTo Schema
3. **实体优先** — 开头明确命名主体（如 Celsius、Fahrenheit），名称全文一致
4. **可引用数据** — 引用权威来源（WHO/USDA/NOAA），提供精确数字
5. **FAQ Schema** — 每个问答 40-60 字，用 JSON-LD 标记
6. **时效性** — 标注最后更新日期，定期刷新

## 技术实现
- 旧文件不修改，新页面用新 URL 路径
- 新组件放在 `components/` 下新建文件
- 新页面放在 `pages/` 下，与旧页面同级
- 如果新页面 URL 与旧页面冲突，直接替换旧文件内容
- 不做程序化模板，不做批量生成
- 每个交互组件独立编写，不复用旧站模板逻辑

## 禁止行为
- ❌ 使用 textSpinner.ts 或任何同义替换工具
- ❌ 用相同页面结构换数字（如 10-c-to-f、20-c-to-f 模式）
- ❌ 关键词堆砌
- ❌ 广告嵌入内容流（广告放在内容之外）
- ❌ 修改旧文件（包括旧页面、旧组件、旧配置）
- ❌ 批量创建页面

## 维护
- 定期检查 Google Search Console 收录状态
- 根据收录数据调整内容策略
- 旧页面逐步添加 noindex 或 301 到新站对应页面
- 新站流量稳定前，保持旧站完整在线

---
> Source: [Jovceo/ctofconverter](https://github.com/Jovceo/ctofconverter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
