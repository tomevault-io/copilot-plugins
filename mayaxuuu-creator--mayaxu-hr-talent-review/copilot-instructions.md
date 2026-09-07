## mayaxu-hr-talent-review

> 纯前端单文件人才盘点工具（绩效×潜力九宫格）。零依赖、零框架、零后端，数据存 localStorage。

# AGENTS.md — Maya's Talent Review · AI 协作规则

## 项目定位

纯前端单文件人才盘点工具（绩效×潜力九宫格）。零依赖、零框架、零后端，数据存 localStorage。

## 怎么跑

```bash
node src/build_platform.js      # 中文版：从 src/engine.js 构建
node src/build_platform.en.js   # 英文版：从 src/engine.en.js 构建
```

构建后手动复制：
- 中文：根目录 `talent_review_platform.html`（测试）+ `dist/index.html`（Vercel）
- 英文：根目录 `talent_review_platform.en.html`（测试）+ `dist/en/index.html`（Vercel）

## 技术栈

纯 JavaScript + HTML + CSS。构建脚本 `src/build_platform.js` 是 Node.js，无第三方依赖。

## 目录与约定

- **唯一源码**：`src/engine.js`（中文）/ `src/engine.en.js`（英文），全部逻辑 ~2450 行
- **构建脚本**：`src/build_platform.js`（中文）/ `src/build_platform.en.js`（英文）
- **成品**：`dist/index.html`（中文 Vercel）、`dist/en/index.html`（英文 Vercel）、根目录双版本（本地测试）
- **测试数据**：`test-data/`（评估人/被评估人/关键岗位 CSV + 5 个团队评估结果 + 工作流指南）
- **文档**：`docs/`（PRD.md, 使用说明.md），根目录 `CHANGELOG.md`
- **归档**：`legacy/`（v0.9 双工具末期版本）
- 请勿在根目录放源码副本或成品副本——源码只在 `src/`

## 数据模型

- `employees[]` 被评估人员，`eltList[]` 评估人，`keyPositions[]` 关键岗位
- 三实体通过 `mgrId`（员工→评估人）和岗位名称（岗位→负责人）关联
- `csvData[]` 回收的评估结果，`successionData{}` 以岗位名称为键
- localStorage：`trp_data_v1`（数据）、`trp_config_v1`（配置）、`trp_assessments_v1`（评估）、`trp_dev_pwd`（密码）、`trp_guide_done_*`（引导标记）

## 构建与部署

```bash
node src/build_platform.js                                      # 构建中文版
cp src/talent_review_platform.html talent_review_platform.html   # 中文本地测试
cp src/talent_review_platform.html dist/index.html               # 中文 Vercel 部署
node src/build_platform.en.js                                   # 构建英文版
cp src/talent_review_platform.en.html talent_review_platform.en.html  # 英文本地测试
cp src/talent_review_platform.en.html dist/en/index.html              # 英文 Vercel 部署
```

Vercel 项目 `talent-review-private`，从 GitHub `main` 分支部署 `dist/index.html`。
CLI 预览：`VERCEL_CACHE_DIR=/private/tmp/vercel-cache vercel deploy`

## 当前状态

- **V4.0 已上线**（production: talent-review-private.vercel.app，含英文版 /en）
- 离职风险：百分比阈值（综合分 ≥50% 高风险，25-50% 中风险，<25% 低风险）
- HRBP 模板：评分引导页 + 数据录入页两个 CSV
- 新手引导：项目主理人和评估人首次登录自动触发，侧边栏有重播按钮
- 中英双语：中文版 `dist/index.html`，英文版 `dist/en/index.html`
- 开源许可证：MIT License（品牌名称受 TRADEMARK.md 保护）
- 下一步待定（按用户实测反馈迭代）

---
> Source: [mayaxuuu-creator/MayaXu-HR-Talent-Review](https://github.com/mayaxuuu-creator/MayaXu-HR-Talent-Review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
