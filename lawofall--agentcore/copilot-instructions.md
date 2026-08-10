## cursor-deploy

> 全端部署（后端+官网+手机+管理后台+桌面）——用户说「部署/上线/发版」时读；凭据存 deploy/.env.deploy.local 不入仓。


# 部署规范

完整操作手册 → [`发布与门禁.md` · 本地发布操作手册](/docs/05-平台与运维/发布与门禁.md#本地发布操作手册)。入口壳 → [部署与运维](/docs/05-平台与运维/部署与运维.md)。

## 铁律

- **凭据禁令**：`deploy/.env.deploy.local` 不入仓。禁止在 chat 索要/回显 token、SSH 私钥或其它部署凭据。
- **不重新启用** deploy/release workflow 的 push/tag/schedule 自动触发。
- **新增/改接口 → 后端必须先部署**。前端 deploy 脚本内置版本漂移门禁。
- **桌面发版**：Win = 本地 `release:win`（`--publish never` + `gh upload` + 资产断言 + `sync:release-cdn`；对齐 Mac）；Mac = GHA `Release Desktop`（仅 mac）。为省额度：GHA 桌面发布仅 Mac；Win 本地；**勿用 GHA 打 Win**。官网首装 = GitHub Releases；updater feed = 品牌域 nginx（见 [发布与门禁 §7.6b](/docs/05-平台与运维/发布与门禁.md)；勿在 chat 回显主机名或凭据）。
- 发布前本地一键门禁：`pnpm release:gate`（与 `ci.yml` 同构；触发保留，不关 CI）。含 **schema gate**（迁移 head ↔ ORM；`uv run python scripts/check_schema_gate.py`）。日常迭代可用 `pnpm release:gate:lite`（跳过 desktop shoot + smoke；**发布仍须全量**）。
- **后端上线顺序**：`finish-server.sh` / `deploy-server.sh` = 停旧 api → `alembic upgrade` → schema gate `--live` → 起新 api。禁止旧容器接流量时跑破坏性迁移。

## 全端发布顺序（速查）

半自动清单（只打印步骤/探针，不替你 Publish）：`pnpm release:ship`（`--track api` = 仅热修轨；`--check` = 探桌面·Android draft 资产）。full 清单含 Android APK 步（`release:android` → 冒烟 → Publish `android-v*`）。

**公告两段式（定案 D · 工作流 A）**：人定「今天发 + 约时」后**立刻预告**（与 gate / 修拦阻并行）；门禁不过则改约时或归档预告。收口在验收完成后发。勿绑进 `deploy:backend` 自动发。**部署前**仍须 `release:gate` 全绿。

```text
pnpm release:notice -- --phase preview …   # 定案约时后立刻（可与下行并行）
pnpm release:gate                          # Win 默认串行；红灯不得部署
# bump → commit → push
pnpm deploy:backend <sha>
# full：Win release:win ∥ Mac GHA → 齐资产后 Publish
# full：Android release:android → 真机冒烟 → Publish android-v*（CDN 由脚本末尾 sync；与桌面 v* 分轨）
# full：mobile / admin / web →（桌面 Publish 后）website deploy:pages
pnpm release:notice -- --phase done …      # 收口
# tag prod-* / desktop-v*
```

手搓顺序仍等价：git push → 后端 `pnpm deploy:backend <sha>` →（若桌面已 bump：先 Publish 桌面 release）→ Android APK（见上）→ 官网 → 手机 → admin → web 客户端 → 验收 → tag。

公告 CLI 细节 → [`产品公告文案模板`](/docs/05-平台与运维/产品公告文案模板.md) · `pnpm release:notice --help`。

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
