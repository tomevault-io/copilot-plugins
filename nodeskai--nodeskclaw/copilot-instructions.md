## nodeskclaw

> K8s 多集群操作规范：上下文选择与敏感操作确认


# K8s 集群操作规范

## 多集群上下文选择

**本地 kubeconfig 配置了多个集群上下文，执行任何 kubectl 命令前必须确认目标集群。**

### 必须做

- 执行 kubectl 前先 `kubectl config get-contexts` 确认当前上下文和可用集群列表
- 每条 kubectl 命令显式指定 `--context <context-name>`，不依赖 current-context 默认值
- 不确定该操作哪个集群时，先问用户

### 严禁

- **禁止盲用 current-context**：不确认就直接跑 kubectl，可能操作到错误集群
- **禁止 `kubectl config use-context` 切换全局上下文**：会影响用户其他终端，一律用 `--context` 参数
- **禁止假设集群**：即使上次操作的是 A 集群，这次也要重新确认

## 敏感操作确认

**以下操作必须提前向用户确认目标集群和具体内容，获得明确许可后才执行：**

- 删除资源：`kubectl delete`（namespace、deployment、pod、service、ingress、secret、configmap 等）
- 扩缩容：`kubectl scale`
- 滚动更新：`kubectl rollout restart`、`kubectl set image`
- 执行进入容器：`kubectl exec`（写操作，如修改文件、杀进程）
- 应用变更：`kubectl apply` / `kubectl patch`（修改现有资源）

### 确认格式

```
即将对集群 [context-name] 执行以下操作：
1. [具体操作描述]
2. ...

请确认是否执行。
```

### 无需确认的只读操作

- `kubectl get`
- `kubectl describe`
- `kubectl logs`
- `kubectl top`
- `kubectl exec -- cat/ls/printenv` 等只读命令

## 部署操作必须同时确认 context 和 namespace

**部署脚本（`deploy/cli.sh`）和 kubectl 操作必须同时明确目标环境，缺一不可。**

同一个集群上可能同时存在生产和 staging 环境（不同 namespace），仅确认 context 不等于确认了目标环境。

### 已知环境映射

| 环境 | context | namespace |
|------|---------|-----------|
| 生产 | `nodesk-infra-vke-dev-dmz-01` | `nodeskclaw-system` |
| staging | `nodesk-infra-vke-dev-dmz-01` | `nodeskclaw-staging` |
| 实例集群 | `nodeskclaw-inst-dev-01` | 按实例动态分配 |

### 必须做

- 给用户提供部署命令时，**必须同时写明 `--context` 和 `--namespace`**，不依赖脚本默认值
- 部署前先用 `kubectl get namespaces --context <ctx>` 确认目标 namespace 存在
- 不确定用户要部署到哪个环境时，先列出可用环境（context + namespace 组合）让用户选择

### 严禁

- **禁止在需要生产部署时省略 `--prod`**：`cli.sh deploy` 默认部署到 staging，生产需显式 `--prod`
- **禁止假设环境**：用户说"staging"就查 staging namespace，说"生产"就查生产 namespace，不要自己猜
- **禁止只确认 context 不确认 namespace**：同一 context 下有多个环境，只确认 context 不够

### 部署命令示例

```bash
# staging 部署（默认环境）
./deploy/cli.sh deploy backend

# 生产部署（需用户明确授权）
./deploy/cli.sh deploy backend --prod
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NoDeskAI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NoDeskAI)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
