## krkr2web

> 给 AI 编码代理与后续维护者的硬性约定。与用户既有指令冲突时，以用户当次指令为准。

# AGENTS.md — 仓库工作约定

给 AI 编码代理与后续维护者的硬性约定。与用户既有指令冲突时，以用户当次指令为准。

## 一、实机验证服务端（硬性要求）

需要真机（TB-X802F / 腾讯 X5 WebView，Chromium 89）联测时，**必须在 Windows
宿主机的热点 IP `192.168.137.1:443` 上开启产品 + 诊断服务端**，这是用户指定的
真机访问入口：

```powershell
python tools/https_diagnostics_server.py --bind 192.168.137.1 --port 443 `
  --directory build/d5-product --reports-directory diagnostic-reports `
  --cert certs/server.pem --key certs/server-key.pem `
  --resource-map build/d5-product/resource-map.json --auto-catalog
```

- 192.168.137.1 是 Windows 移动热点的默认网关 IP；热点未开启时该网卡不存在，
  先开热点再起服务。
- 必须 HTTPS/443：目标 WebView 不允许混合内容。自签名证书下设备首次访问
  `https://192.168.137.1/` 需接受证书警告；正式流程（DNS 劫持 + 受信 CA）见
  [构建与部署](docs/04-build-and-deploy.md)。
- `resource-map.json` 在服务端启动时**一次性加载**：`prepare_d5_product`
  重新生成产品后必须重启服务端，否则 XP3 按需供给仍用旧映射。
- 诊断报告由设备 `POST /api/diagnostics-reports` 落盘到 `diagnostic-reports/`，
  真机结论以报告为准（该目录不入 git，结论沉淀进 `docs/`）。

## 二、代码与补丁纪律

- **游戏资源绝不入 git**：《千恋万花》（senren-banka）的任何原始文件、解包
  产物、报告原文都不得被跟踪（`参考资料/`、`diagnostic-reports/` 均在
  `.gitignore`）。
- 引擎改动一律以 `engine/patches/NNNN-描述.patch` 提交并在
  `engine/upstream.json` 注册；`build/d3-engine-work/` 是导出的工作树，
  禁止直接改动后不回写补丁。补丁必须能在干净导出树上应用。
- Web 侧改动不得依赖 JSPI / OPFS / SharedArrayBuffer / Service Worker /
  Wasm 原生异常 / SIMD——目标 WebView（Chromium 89）不支持。
- 每轮验证最小化：跑通即直接提交，统一审计留待阶段收尾（本次收尾见
  `docs/09-verification-log.md` 2026-08-25 条目）。

## 三、同步与发布

- WSL 侧真正的工作克隆在 **`~/krkr2web`**（注意：`/mnt/d/Code/FuckChangyan/
  krkr2web` 是 Windows 同一仓库的挂载视图，向它 `git am` 会自我冲突）。同步
  方式：Windows 侧 `git format-patch <区间> -o _wslsync`，然后
  `MSYS_NO_PATHCONV=1 wsl sh -c 'git -C ~/krkr2web -c user.name=aurumple
  -c user.email=aurumple@users.noreply.github.com am /mnt/d/Code/FuckChangyan/
  krkr2web/_wslsync/*.patch'`，完成后核对两侧 `HEAD^{tree}` 一致。
- **未经用户明确要求不 push 到 GitHub。**

---
> Source: [aurumple/krkr2web](https://github.com/aurumple/krkr2web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
