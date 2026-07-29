## arona-webui

> > Arona WebUI coding and workflow conventions.

# Project Conventions

> Arona WebUI coding and workflow conventions.

---

## Project Identity

- **Name**: Arona WebUI (internal: `openclaw-mvp`)
- **Type**: Single-page admin dashboard for OpenClaw AI gateway
- **Stack**: Node.js (raw `node:http`) + vanilla JS (no framework, no build step)

---

## Commands

```bash
npm install     # Install dependencies
npm start       # Run server (or: node src/server.mjs)
```

No tests or lint scripts exist in this project.

---

## Git Workflow

- Single `main` branch, local merge, push to remote
- Commit messages: English verb prefix + Chinese description
  - Examples: `feat(chat): 新增Playground对话`, `fix(auth): 恢复登录页`
- Merge commits: `merge(main): 合并 feat/xxx`

---

## Code Style

- Backend: single file `src/server.mjs`, ESM (`import`/`export`)
- Frontend: vanilla JS ES modules in `public/`, no transpilation
- No TypeScript — plain JavaScript throughout
- Chinese-language UI strings

---

## External Codebase

- **OpenClaw 网关源码**: `/home/lingshi/workspace/openclaw/openclaw-repo/`
- 涉及后端改动时，必须先查阅网关源码确认 RPC 方法、配置结构和业务逻辑（详见 `backend/architecture.md`）
- GitNexus 索引和 ABCoder AST 解析均应覆盖此目录，供 MCP 工具查询

---

## Deployment

- Server on `127.0.0.1:18790`
- Reverse-proxied by Nginx with BasicAuth for public access
- Production reads `/root/.openclaw/openclaw.json` for gateway config fallback

---
> Source: [lingshichat/Arona_WebUI](https://github.com/lingshichat/Arona_WebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
