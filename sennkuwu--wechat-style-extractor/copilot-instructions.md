## wechat-style-extractor

> - 本项目为公众号文章样式提取工具；沿用现有 Vinext、React 和 Sites 托管结构。

# 项目工作约定

## 范围与目录
- 本项目为公众号文章样式提取工具；沿用现有 Vinext、React 和 Sites 托管结构。
- `app/` 放页面、样式与 API 路由；`lib/` 放提取逻辑；`test/` 放回归测试；`public/` 放静态资源。
- 发布与恢复说明放 `README.md`。本地临时检查记录放已忽略的 `work/`，构建产物和缓存不提交；清理文件前先征求用户确认。
- 保留现有未提交改动，不做与任务无关的重构。

## 验证
要求 Node.js >=22.13.0。发布前运行：

```bash
npm test
npm run lint
npx tsc --noEmit --incremental false
npm run build
git diff --check
```

页面改动还需检查桌面、窄屏、文章提取、错误提示和复制反馈。明确记录无法执行的验证，不把已有记录当成本次结果。

## 安全与发布
- 不读取或提交 `.env`、密钥、token、密码；不提交构建产物与本地缓存。
- GitHub 与 Sites 是独立远端；更新 GitHub 不代表体验站已部署。
- 发布前核实远端与线上旧版本，保留可恢复的旧版代码引用。
- Git push、生产发布、CI/CD 或托管配置修改必须先单独征求用户确认。
- 不删除文件或 Git 历史，不 rebase、reset --hard 或强推；此类操作必须先征求用户确认。
- 保留各远端提交历史，使用正常快进或经审查的提交移植同步。

---
> Source: [sennkuwu/wechat-style-extractor](https://github.com/sennkuwu/wechat-style-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
