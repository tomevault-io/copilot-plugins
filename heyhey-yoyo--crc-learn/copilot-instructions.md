## crc-learn

> CRC Evidence Lab 是一个面向准博士生和青年研究者的**双案例临床试验分步学习课程**网页，纯静态多文件实现（`index.html` + `styles.css` + `app.js`），零依赖、零构建，目标部署平台为 Cloudflare Pages。

# CRC Evidence Lab — 项目说明（供 AI 编程代理阅读）

## 项目概览

CRC Evidence Lab 是一个面向准博士生和青年研究者的**双案例临床试验分步学习课程**网页，纯静态多文件实现（`index.html` + `styles.css` + `app.js`），零依赖、零构建，目标部署平台为 Cloudflare Pages。

## 项目结构

| 文件 | 作用 |
| --- | --- |
| `index.html` | 页面结构（HTML），引用 `styles.css` 与 `app.js` |
| `styles.css` | 全部样式（原单文件内 `<style>` 抽取，约 52 KB） |
| `app.js` | 全部交互逻辑与课程内容数据（原单文件内 `<script>` 抽取，约 485 KB） |

## 技术栈与运行架构

| 技术 | 说明 |
| --- | --- |
| HTML / CSS / JavaScript | 多文件静态网页，无框架、无打包器、零 npm 依赖 |
| localStorage | 学习进度本地持久化，键前缀请保持稳定 |
| Cloudflare Pages | 静态托管，构建命令留空，构建输出目录为 `/` |

## 本地运行

```bash
python -m http.server 8000
```

浏览器打开 `http://localhost:8000`。也可直接双击 `index.html` 在浏览器中打开（无构建、无依赖）。

## 代码组织与风格约定

- **结构**：HTML 在 `index.html`，样式在 `styles.css`，逻辑与数据在 `app.js`，三者职责分离
- CSS 变量定义在 `:root`，统一视觉令牌（颜色、圆角、阴影）
- 中文界面文案，标识符使用英文
- 修改版本号时，同步更新 `<title>`、页面品牌标识与 README
- 保持零依赖原则，未经明确批准不得引入外部库或构建工具
- **已清理的外部引用**：拆分时移除了原单文件中的 `/Static/`、`/Web/` 外部 CSS/JS 引用（Cloudflare Pages 上不存在、会 404，且代码不依赖它们）；Cloudflare beacon 脚本保留

## 部署

**Cloudflare Pages（GitHub 集成）**

```text
Production branch: main
Build command: （留空）
Build output directory: /
```

**直接上传**

```bash
npx wrangler pages deploy . --project-name crc-learn
```

## 安全与数据注意事项

- 所有数据保存在浏览器 `localStorage`，不上传服务器
- 无后端、无身份验证、无多用户支持
- 不包含用户数据采集逻辑

---

## AI 维护提醒

> **⚠️ 任何修改此项目的 AI 代理（包括未来的你自己）都必须遵守：**
>
> - **修改代码后必须同步更新本 AGENTS.md 与 README.md** — 功能增删、版本变更、部署方式变更都需要在两份文档中体现
> - README.md 面向**人类用户**，AGENTS.md 面向 **AI 代理**，两份文件**不可互相替代**
> - 项目为多文件结构（`index.html` + `styles.css` + `app.js`），新增/拆分文件时务必同步更新文件清单

---
> Source: [heyhey-yoyo/CRC-learn](https://github.com/heyhey-yoyo/CRC-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
