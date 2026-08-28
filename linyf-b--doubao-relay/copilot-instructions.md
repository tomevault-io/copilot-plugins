## doubao-relay

> 本地豆包网页版中转，**无 Docker**，仅限个人自用。

# Agent instructions — Doubao Relay

本地豆包网页版中转，**无 Docker**，仅限个人自用。

## 启动

```bash
copy .env.example .env
npm run setup
npm start
```

- 管理页 http://127.0.0.1:8787 →「登录豆包」
- 上游 `vendor/doubao-free-api` 本机 Node `:8000`

勿提交 `data/`、`.env`、`vendor/**/node_modules`。

---
> Source: [linyf-B/doubao-relay](https://github.com/linyf-B/doubao-relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
