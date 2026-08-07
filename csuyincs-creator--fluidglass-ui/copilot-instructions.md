## fluidglass-ui

> WebGL 流体玻璃卡片界面生成技能包（agent skill + 零依赖前端参考实现）。设计哲学：**固定视觉内核（Canonical Material Core）+ 开放业务配置**。任何改动不得破坏与 `references/canonical-reference.png` 的视觉一致性。

# AGENTS.md

## 项目定位

WebGL 流体玻璃卡片界面生成技能包（agent skill + 零依赖前端参考实现）。设计哲学：**固定视觉内核（Canonical Material Core）+ 开放业务配置**。任何改动不得破坏与 `references/canonical-reference.png` 的视觉一致性。

## 怎么跑起来

```bash
# 看效果（无需构建/服务器）
start references/canonical-demo-single-file.html
# 质量门禁（改动后必须过）
python tools/package_qa.py                    # 必须 "passed": true
python tools/package_qa.py --update-hashes    # 仅改内核文件后
node --check references/fluid-glass-engine.js
```

## 技术栈

- 纯原生 JS + WebGL1，零依赖：无 CDN、无外部字体、无网络请求
- CSS 降级渲染；`localStorage` 持久化；导出 HTML
- 文档默认中文；README 双语：`README.md`（英文）+ `README.zh-CN.md`（中文），顶部互链；`docs/demo.gif` 为 README 头图（由 headless Chrome 截帧 + ffmpeg 合成，约 437KB/2s）

## 目录与约定

- `references/` = 不可变视觉内核（shader / css / engine / demos / reference.png）。改内核 = 同步 `CANONICAL_CORE_HASHES.json` + 重生成对照截图 + 过 QA
- `docs/brand/` = 品牌资产（小策角色形象，**保留版权，不随 Apache-2.0 授权**）——**禁止**把这里的文件纳入 Apache 授权范围、禁止改作/转授权/加入训练数据；改动需用户本人确认。`docs/brand/README.md` 是品牌声明唯一权威来源
- `SKILL.md` = Agent 技能入口（YAML frontmatter 的 `name`/`description` 勿删）；`manifest.json` 的 `name` 必须与之一致
- `tools/package_qa.py` = 包完整性 + 内核自检（无第三方依赖）
- 卡片上限 12；WebGL 上下文预算 8（超出走 CSS fallback）——这是刻意设计，不是 bug
- JS 风格：2 空格、单引号、分号、不引入构建依赖；文档中英文与数字间加空格

## 当前状态和下一步

- 版本 1.0.0，Apache-2.0（品牌资产 `docs/brand/` 例外，保留版权）；5 个 P0 缺陷已修复并完成 7 场景 headless Chrome 回归（dupkey 死循环 / NaN / XSS / 坏 localStorage / 持久化 / 12 卡预算）
- 仓库已 `git init` 并推至 https://github.com/csuyincs-creator/fluidglass-ui（private）；README 双语 + 动效 GIF 已上线；`.github/workflows/qa.yml` 待 workflow-scoped token 补推
- 贡献规则详见 `CONTRIBUTING.md`；合入前请逐条对照 `ACCEPTANCE_CHECKLIST.md`

---
> Source: [csuyincs-creator/fluidglass-ui](https://github.com/csuyincs-creator/fluidglass-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
