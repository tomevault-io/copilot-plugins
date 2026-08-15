## nas-deployment-public

> 本仓库是 `gtr` 服务器及远程 edge/proxy 节点的**基础设施 runbook 与部署自动化**仓库。不含应用源码——只有 Ansible playbook、Shell 脚本、配置模板、监控面板和文档。

# AGENTS.md

本仓库是 `gtr` 服务器及远程 edge/proxy 节点的**基础设施 runbook 与部署自动化**仓库。不含应用源码——只有 Ansible playbook、Shell 脚本、配置模板、监控面板和文档。

所有文档使用 **中文**。

## 核心规矩

1. **无明文凭据** — 敏感值分两层管理：
   - **节点层**（Ansible host_vars/group_vars）→ SOPS + AGE 加密，存入私有 `nas-deployment-vault`，`.runtime.yml` gitignored
   - **K8s 层**（集群内 Secret）→ `kubeseal` 加密为 SealedSecret CRD，直接提交到本仓库 `platform/resources/`，由 ArgoCD 同步
   
   详细加密操作见 [`docs/deployment/secrets-encryption-guide.md`](docs/deployment/secrets-encryption-guide.md)。
   Vault 仓库结构见 [`docs/deployment/secrets-management.md`](docs/deployment/secrets-management.md)。

2. **Ansible 约定** — 详见 [`docs/topic/infrastructure/ansible-conventions.md`](docs/topic/infrastructure/ansible-conventions.md)。关键：assert > fail，notify + flush_handlers，CI 中 shell task 用 pipefail 必须加 `args: executable: /bin/bash`

3. **K3s 应用由 ArgoCD 管理** — 所有 K3s 集群内应用（监控、日志、代理、面板等）必须通过 ArgoCD GitOps 部署，禁止手动 `kubectl apply` 或 Ansible 直接部署 K8s 资源。ArgoCD 自身可由 Ansible 引导安装（唯一例外）。Ansible 仅用于基础设施层（Tailscale、K3s、内核模块、Mihomo 等）。详见 [`docs/planning/migrate-services-ansible-to-k3s.md`](docs/planning/migrate-services-ansible-to-k3s.md)。

4. **K3s 平台 CI 部署** — ArgoCD + SealedSecrets + Tailscale Operator 通过 GitHub Actions 自动部署。详见 [`docs/deployment/k3s-platform-ci-deployment-guide.md`](docs/deployment/k3s-platform-ci-deployment-guide.md)

5. **Pre-commit** — 提交前自动运行 `actionlint`（workflow lint）、`check-yaml`、明文凭据守卫等。安装：`bash scripts/setup-git-hooks.sh`。若有 `pre-commit` 则体验更完整：`pip install pre-commit && pre-commit install`

## 仓库结构

```
nas-deployment-public/
├── edge/ansible/             # Ansible playbooks, roles, host_vars（核心）
├── platform/                 # K3s 内平台组件（ArgoCD GitOps source）
│   ├── applications/         # ArgoCD Application CRDs
│   ├── helm-values/
│   └── resources/            # 直接应用的 K8s 资源（如 ProxyClass）
├── k3s/                      # K3s 平台文档
├── mihomo/                   # Mihomo 代理客户端（Ansible）
├── cigbutt/                  # 量化分析 CLI（Python, hatchling）
├── victoriatraces/           # VictoriaTraces 分布式追踪
├── network-monitor/          # 网络连通性监控
├── grafana/                  # Grafana 面板、告警
├── envoy/ victoriametrics/ victorialogs/ node_exporter/  # 各服务 runbook
├── .github/                  # CI/CD workflows + actions
├── scripts/
│   └── check-no-plaintext-secrets.py
└── docs/
    ├── deployment/           # 部署指南、CI guide、secrets management
    ├── topic/infrastructure/ # Ansible 约定等专题
    ├── troubleshooting/      # 排错分析
    └── issues/               # 实现跟踪（gitignored）
```

## 基础设施拓扑

```
K3s Cluster
├─ tencent (control-plane): Tailscale 100.99.48.76
├─ gtr (agent):            100.121.0.67
└─ aliyun (agent):         100.100.99.70
API: https://100.99.48.76:6443
Pod CIDR: 10.60/16, Service CIDR: 10.61/16, Flannel: vxlan (tailscale0)

GTR Core (192.168.31.59)
├─ Envoy → Grafana :3000 / VictoriaMetrics :8428 / VictoriaLogs :8429 / VictoriaTraces :9428
└─ Mihomo :7890（shadow-tls SIP003 代理）

Edge Nodes (remote_proxy, aliyun)
├─ Envoy :80/:443 + Node Exporter
└─ Vector → OTLP → forwards logs/traces to GTR via Tailscale

K3s Server (tencent)
└─ Envoy :80/:443 + Node Exporter

Remote Proxy (66.154.100.187)
└─ Shadow-TLS :443 (SNI: www.microsoft.com) → Shadowsocks :8388
```

### K3s Platform（已部署）

| Component | ArgoCD UI | Tailscale 访问 |
|-----------|-----------|---------------|
| **ArgoCD** | `https://argocd-argocd-server.tail414c32.ts.net` | ProxyClass gtr-only |
| **Sealed Secrets** | 集群内 | — |
| **Tailscale Operator** | 集群内 | 服务暴露 `tailscale.com/expose: "true"` |

## 常用部署命令

```bash
# === K3s 集群 ===
cd edge/ansible
ansible-playbook -i inventory-edge.ini deploy-gtr-k3s-server.yml
ansible-playbook -i inventory-edge.ini deploy-gtr-k3s-agent.yml

# === Platform Operators ===
ansible-playbook -i inventory-edge.ini deploy-platform-argocd.yml
ansible-playbook -i inventory-edge.ini bootstrap-platform-sealed-secrets-key.yml
ansible-playbook -i inventory-edge.ini deploy-platform-tailscale-operator.yml

# === 验证 ===
ansible-playbook -i inventory-edge.ini verify-platform-argocd.yml
ansible-playbook -i inventory-edge.ini verify-platform-sealed-secrets.yml
ansible-playbook -i inventory-edge.ini verify-platform-tailscale-operator.yml

# === Edge ===
ansible-playbook -i inventory-edge.ini deploy-edge.yml

# === 完整 CI 触发 ===
gh workflow run deploy-infra.yml --ref main -f target=all
```

## 关键远程路径

| Path | Purpose |
|------|---------|
| `/etc/envoy/` | Envoy bootstrap + dynamic configs |
| `/usr/local/grafana/` | Grafana |
| `/var/lib/victoriametrics/data` | Metrics storage |
| `/var/lib/victorialogs/data` | Log storage |
| `/var/lib/rancher/k3s/` | K3s data + SealedSecrets 密钥备份 |
| `/etc/vector/` | Vector config |

## Gotchas

1. **`remote_proxy` 已加入 K3s agent（2026-06）：** 打 `NoSchedule` taint，仅运行 DaemonSet（node_exporter、Vector）。跨洋 ~200ms 延迟，不调度业务 Pod。详见 [`docs/planning/migrate-services-ansible-to-k3s.md`](docs/planning/migrate-services-ansible-to-k3s.md)。

2. **Tunnel client 已从 GTR 移除:** Mihomo 内置 shadow-tls SIP003 处理所有代理流量。

3. **K3s 禁用了 servicelb + traefik:** 服务暴露走 Tailscale Operator。

4. **Tailscale proxy 调度:** 用 `ProxyClass gtr-only` 强制到 GTR（tencent 有 WireGuard TCP 阻断）。

5. **YAML name 冒号陷阱:** GitHub Actions 的 `name:` 值含 `: ` 必须加引号（如 `name: "Manual: backup"`），否则 YAML parser 误解析。

6. **Cigbutt:** Python CLI 量化分析，位于 `cigbutt/`。详见 `cigbutt/README.md`。

## 文档索引

| 主题 | 文档 |
|------|------|
| Ansible 约定 | [`docs/topic/infrastructure/ansible-conventions.md`](docs/topic/infrastructure/ansible-conventions.md) |
| Secrets 加密操作 | [`docs/deployment/secrets-encryption-guide.md`](docs/deployment/secrets-encryption-guide.md) |
| Secrets 架构与 Vault | [`docs/deployment/secrets-management.md`](docs/deployment/secrets-management.md) |
| K3s 平台 CI 部署 | [`docs/deployment/k3s-platform-ci-deployment-guide.md`](docs/deployment/k3s-platform-ci-deployment-guide.md) |
| K3s 平台概述 | [`k3s/README.md`](k3s/README.md) |
| Vault 仓库 | `nas-deployment-vault/README.md`（私有仓库） |
| 排错文档 | [`docs/troubleshooting/`](docs/troubleshooting/)

---
> Source: [Lynskylate/nas-deployment-public](https://github.com/Lynskylate/nas-deployment-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
