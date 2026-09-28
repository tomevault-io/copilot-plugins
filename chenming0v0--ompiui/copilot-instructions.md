## ompiui

> - 默认在 `dev` 工作。`dev` 是集成分支：功能 PR 进这里，维护者也可以直接推送。

# AGENTS.md

## 分支

- 默认在 `dev` 工作。`dev` 是集成分支：功能 PR 进这里，维护者也可以直接推送。
- `main` 是发布分支：只接受本仓库 `dev` 合并进来的 PR。不直接推送，不把功能 PR 开到 `main`。
- 开 PR 时 base 只能是 `dev`。唯一例外是用户明确要求把 `dev` 合并进 `main`。

## 不在 dev 时

当前分支不是 `dev`，且用户没有明确要求留在该分支：先说明当前分支并告知要切到 `dev`，执行 `git checkout dev`，然后继续工作。

## 版本

- 从 `0.1.0` 起。根包、`app` / `server` / `omp-worker` / `protocol`、Tauri 用同一个号。
- 发版记录写根目录 `CHANGELOG.md`。`packages/app/CHANGELOG.md` 是上游 PiUI 历史，不参与发版。
- 升版本：`npm run release:prepare -- <version>`，或 `npm run release:bump -- <version>`。提交留在 `dev`，合并进 `main` 之后再打 tag：

```bash
git fetch origin main
git tag v<version> origin/main
git push origin v<version>
```

- `v*` tag 触发 Desktop And Mobile Release。Android 签名 secrets：`ANDROID_KEYSTORE_BASE64`、`ANDROID_KEYSTORE_PASSWORD`、`ANDROID_KEY_ALIAS`。
- 应用内更新检查读本仓库 `releases/latest`，不是 PiUI。

---
> Source: [chenming0v0/OMPiUI](https://github.com/chenming0v0/OMPiUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
