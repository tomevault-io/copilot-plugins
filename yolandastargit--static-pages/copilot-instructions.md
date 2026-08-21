## static-html-publish

> 用户扔 HTML 原件时，自动发布到 GitHub Pages 并返回分享链接


# 静态 HTML 自动发布

用户发送 HTML 文件、粘贴 HTML 内容，或说「发布」「上线」「分享链接」时，**立即执行发布**，不要只给步骤。

## 固定配置

| 项 | 值 |
|---|---|
| 本地目录 | `/Users/yolandaw-/Documents/仓库` |
| GitHub | `yolandastarGit/static-pages` |
| 基础 URL | `https://yolandastarGit.github.io/static-pages` |
| 分支 | `main` |

## 文件路径规则

按项目分文件夹，**禁止中文和空格**：

```
pages/{项目slug}/index.html      ← 整个系统/主原型
pages/{项目slug}/{页面}.html     ← 子页面
```

| 用户说的项目 | slug |
|---|---|
| 苏豪 | `suhao` |
| 未说明 | 从项目名转拼音小写，或问一句 |

## 发布流程

1. 保存 HTML 到 `pages/{slug}/...`
2. `git add` → `git commit -m "publish: {slug}/{文件名}"` → `git push origin main`
3. 返回链接（markdown 可点击）：
   - 主页面：`{基础URL}/pages/{slug}/`
   - 子页面：`{基础URL}/pages/{slug}/{页面}.html`
4. 提醒部署约 1–2 分钟

## 禁止

- 不改 `.github/workflows/deploy-pages.yml`
- 不用中文/空格作路径名
- push 失败时给出终端命令，不要静默失败

---
> Source: [yolandastarGit/static-pages](https://github.com/yolandastarGit/static-pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
