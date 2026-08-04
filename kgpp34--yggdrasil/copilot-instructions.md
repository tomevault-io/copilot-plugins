## yggdrasil

> 面向多集群、多租户、多负载形态的统一发布平台。前期只做**在线服务**(无状态+有状态),离线和 Agent 负载留扩展点以后做。

# Yggdrasil — 基于 Kubernetes 的统一发布平台

## 项目概述

面向多集群、多租户、多负载形态的统一发布平台。前期只做**在线服务**(无状态+有状态),离线和 Agent 负载留扩展点以后做。

## 开发环境

| 项目 | 信息 |
|---|---|
| Go 版本 | **1.24.4** (darwin/arm64) |
| 本地 K8s | **kind v0.14.0**, 集群 `dev` (K8s v1.28.0, 1 control-plane + 1 worker) |
| kind 配置目录 | `~/Project/k8s/kind/` |
| kubectl | v1.28.4 |
| Helm | v3.11.2 |
| 架构 | arm64 (Apple Silicon) |

### 现有 kind 集群

- `dev` — 当前活跃集群, K8s v1.28.0, 1cp+1worker
- 可以创建更多 kind 集群用于多集群开发测试

## 核心设计原则(必须遵守)

1. **高内聚, 低耦合** — 每个模块独立, 接口清晰, 不互相依赖内部实现
2. **扩展性优先** — KubeVela 以后可能被替换, 所有与 KubeVela 的交互必须通过抽象层/适配器模式, 不允许在业务逻辑中直接调用 KubeVela API
3. **前期只做在线服务** — webservice(无状态) 和 stateful-service(有状态), 离线/Agent 等以后做但要留扩展点(ComponentDefinition 接口/Trait 接口)
4. **所有 K8s 交互用 Golang** — controller-runtime / client-go
5. **模块设计需细化** — 每个模块后续需要补充详细设计文档(docs/ 下), 包括具体功能列表、API 接口、交互流程

## 设计文档索引

| 文档 | 内容 |
|---|---|
| [00-architecture-overview](docs/00-architecture-overview.md) | 概述·需求·总体架构 |
| [01-iam](docs/01-iam.md) | 租户与权限模块 |
| [02-cluster-management](docs/02-cluster-management.md) | 集群管理模块 |
| [03-environment-management](docs/03-environment-management.md) | 环境管理模块 |
| [04-application-management](docs/04-application-management.md) | 应用管理模块 |
| [05-release-orchestration](docs/05-release-orchestration.md) | 发布编排模块(核心) |
| [06-offline-task](docs/06-offline-task.md) | 离线任务模块(以后做) |
| [07-agent-workload](docs/07-agent-workload.md) | Agent 负载模块(以后做) |
| [08-quota-governance](docs/08-quota-governance.md) | 配额治理模块 |
| [09-observability-audit](docs/09-observability-audit.md) | 可观测与审计模块 |
| [10-data-model](docs/10-data-model.md) | 数据模型设计 |
| [11-key-flows](docs/11-key-flows.md) | 关键流程设计 |
| [12-isolation-planning-risks](docs/12-isolation-planning-risks.md) | 隔离·规划·风险 |

## 实现优先级

- **[实现计划与进度](docs/implementation-plan.md)** — 所有任务的当前状态,每次开发前先看
- **Phase 1 (当前)**: MVP 单集群闭环 — 租户/权限、集群管理、环境管理、应用管理(webservice)、发布编排(滚动/分批/回滚)、配额、可观测
- **Phase 2**: 多集群 + 金丝雀/蓝绿 + 有状态服务
- **Phase 3**: 离线任务(以后)
- **Phase 4**: Agent 负载(以后)

## 技术栈选型

- 应用交付: KubeVela(OAM) — **必须通过适配器层封装**
- 工作负载增强: OpenKruise (CloneSet/Advanced StatefulSet)
- 渐进式发布: Kruise Rollouts
- 元数据存储: MySQL/PostgreSQL
- 缓存/锁: Redis
- 消息: Kafka/NATS

## 架构约束

```
接入层(API Gateway)
  → 平台服务层(无状态 Go 服务,持有元数据)
  → 交付引擎适配层(封装 KubeVela,可替换)
  → 成员集群数据面(OpenKruise + Rollout)
```

- 平台服务不直接操作成员集群, 通过适配层翻译为 KubeVela Application CR
- DB 中的发布单与 Revision 是事实源, KubeVela Application 是派生物

---
> Source: [kgpp34/Yggdrasil](https://github.com/kgpp34/Yggdrasil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
