## ssb

> Go 单二进制工具：解析代理分享链接/订阅 → 生成 sing-box 1.13 config.json（TUN+FakeIP+clash_api）→ 托管 sing-box 进程。TUI(bubbletea v1) + CLI 双入口。

# ssb 开发说明（给 Claude Code）

Go 单二进制工具：解析代理分享链接/订阅 → 生成 sing-box 1.13 config.json（TUN+FakeIP+clash_api）→ 托管 sing-box 进程。TUI(bubbletea v1) + CLI 双入口。

## 构建与测试

```bash
# Go 工具链装在 ~/toolchains/go（不污染系统）；本机另有 sing-box 1.13.12 in PATH
export PATH=$HOME/toolchains/go/bin:$PATH
go build -o ssb ./cmd/ssb
go test ./...
gofmt -l .   # 提交前保持干净
```

## 验证（重要）

- 任何 render 改动后：`SSB_DIR=/tmp/ssb-e2e ./ssb add "<链接>"` 会触发 gen + 本机 `sing-box check`——这是 1.13 字段名的最终裁判，不要只信单测。
- 无 root e2e：把 `$SSB_DIR/data/state.json` 的 `tun_enabled` 改 false → `./ssb start` → curl `http://127.0.0.1:9090/version -H "Authorization: Bearer <secret>"` → `./ssb stop`。secret 在 state.json。
- TUN 配置合法性用 `./ssb gen`（check 不需要 root）验证即可；**不要**在这台远程 VM 上真开 TUN/auto_route（会断 SSH）。

## 结构

- internal/link — 分享链接解析器（每协议一文件 + 金样例测试）；build.go 里的 TLSBlock/TransportBlock 被 clash2sb 复用
- internal/sub — 订阅拉取（UA 退避：sing-box→clash.meta→v2rayN）+ 三格式探测
- internal/clash2sb — Clash YAML proxies → sing-box outbound
- internal/profile — state.json（订阅/手动节点/设置）；Dirs 集中管理所有路径
- internal/render — config.json 生成（锁 1.13 语法：新版 DNS type 字段、rule actions、rule-set remote）
- internal/sbx — 内核定位/下载/check/进程管理（setsid+pidfile）/doctor
- internal/app — CLI/TUI 共用的业务层；Generate() 带 check+回滚
- cmd/ssb — 手写子命令分发（无 cobra）；internal/tui — 四页签

## 约定

- 生成的配置永远先 `sing-box check` 再落地（Generate 里已实现，勿绕过）
- 工具绝不改系统文件（resolved 等只打印建议）
- 出错信息用中文，面向终端用户

---
> Source: [onlyghb/ssb](https://github.com/onlyghb/ssb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
