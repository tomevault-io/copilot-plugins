## chzpng

> 这是一个本地运行的图片压缩 Web 工具。保持前端流程直观、后端处理可靠，并避免将用户上传的图片持久化到磁盘。

# Agent Guide

## 项目目标

这是一个本地运行的图片压缩 Web 工具。保持前端流程直观、后端处理可靠，并避免将用户上传的图片持久化到磁盘。

## 运行与验证

- 安装依赖：`npm install`
- 启动服务：`npm start`
- 开发监听：`npm run dev`
- 默认地址：`http://localhost:3000`
- 修改图片处理逻辑后，至少用 `POST /api/compress` 验证一种 PNG 或 JPG 输入和 WebP 输出。
- 修改批量下载逻辑后，验证 `/api/download-zip` 返回以 `PK` 开头的 ZIP 文件。

## 代码边界

- `server.js`：唯一的服务端入口，负责静态资源、上传校验、Sharp 压缩和 ZIP 输出。
- `public/index.html`：页面语义和模板。
- `public/styles.css`：所有页面样式与响应式断点。
- `public/app.js`：浏览器端状态、上传、调用 API、对比渲染和下载。

## 约定

- 支持的 MIME 类型、文件数量和文件大小限制必须同时与前端提示及后端校验保持一致。
- 压缩接口返回的 `compressed.dataUrl` 是前端预览、单张下载和 ZIP 下载的共同数据源，修改其结构时需要同步修改前后端。
- 避免引入前端框架或构建工具，除非需求确实需要；当前项目设计为 `npm start` 后即可运行。
- 新增依赖时更新 `package.json` 与 `package-lock.json`，优先选择维护中的稳定版本。
- 保持 UI 主色调为绿色，并检查 375px 宽度下没有横向溢出。

## 安全要求

- 不要将用户上传文件写入持久目录。
- 不要将 Data URL、文件名或请求参数直接用于文件系统路径；ZIP 文件名需保留 `path.basename` 防护。
- 保留服务器端 MIME 类型、文件数量和文件大小限制；不要只依赖浏览器端校验。

---
> Source: [FengShenNo1/chzpng](https://github.com/FengShenNo1/chzpng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
