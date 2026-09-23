## ad-radar

> 开始任何工作前，先读 [`docs/HANDOFF.md`](docs/HANDOFF.md)。

# 给 AI 助手（Codex / Claude 等）的工作规则

开始任何工作前，先读 [`docs/HANDOFF.md`](docs/HANDOFF.md)。

## 每次改动都必须做

1. **在 [`CHANGELOG.md`](CHANGELOG.md) 最上面加一条记录**：日期、版本、做了什么、为什么、注意事项。没有例外，包括只改配置、只改文案、只做运维操作。
2. **同步更新 `docs/HANDOFF.md`**：改了需求、接口、数据库、配置、服务器、部署方式、已知问题或待办时，更新对应章节和顶部的「最后更新」。
3. **改了插件代码就把 `extension/manifest.json` 的版本号加一**，并在 CHANGELOG 标题里写上版本。
4. **改了提示词（`server/src/prompt.json`）就加对应的版本号**（`version` / `topic_version` / `preset_version` / `ai.version`），否则旧缓存会继续生效。
5. **用户可见的行为变了，就同步官网**（`site/index.html` 的使用说明、常见问题）；**数据收集方式变了，就同步隐私政策**（`site/privacy.html`）。

## 红线

- **这是公开的开源仓库：不要把任何 Jev key、管理员口令、收款私钥、SSH 私钥写进插件、代码仓库、文档、issue 或日志。** 发布包由 `pack.sh` 检查；本机密钥放在 `server/.dev.vars`（已 gitignore）。
- **开源版插件不连我们的服务器**：智能识别用用户自己的 Jev key 直连 `api.typesafe.ai`。不要加回服务端依赖、统计上报或任何把用户内容发到第三方的功能。
- **不要绕过小红书的反爬签名，也不要把抓取挪到服务端。**
- **服务端不保存帖子原文**；任何要存原文的功能先问产品负责人并改隐私政策。
- **改数据库结构前先备份线上库**（命令见 HANDOFF §9.4），并在 `server/src/node.js` 的 `migrate()` 里写迁移。
- **部署后要验证**：`deploy.sh` 末尾会检查接口、官网和安装包；改了 Caddyfile 先在服务器上 `caddy validate`。

## 验证

- 插件：`tests/e2e/`，需要环境变量 `JEV_API_KEY`。
- 旧版服务端（`server/`，只给 1.0 之前的插件用）：`server/test/smoke.mjs`、`test/pay.mjs`、`test/trial.mjs`，改了服务端必须全部通过。
- 官网：`tests/e2e/site-*.mjs`。小红书在无头浏览器会被风控拦截，按 HANDOFF §11.2 的注入方式测，不要尝试绕过风控。
- 动效和界面改动：截图检查，并测减少动效（prefers-reduced-motion）、深色模式、手机宽度。

## 风格

- 代码注释和用户可见文案用中文，和现有代码保持一致。
- 插件不能拖慢页面：空闲时处理、用 `textContent`、不在滚动路径上做重活。
- 动效克制：只动 transform/opacity，自定义 ease-out 曲线，界面动效 < 300ms，hover 只在有鼠标的设备生效。

---
> Source: [pengchujin/ad-radar](https://github.com/pengchujin/ad-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
