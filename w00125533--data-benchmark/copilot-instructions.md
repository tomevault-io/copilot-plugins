## data-benchmark

> - 新增或修改 Docker Compose 基础设施前，必须先检查 `../shared-data-infra` 是否已经定义同类服务或 profile。

# AGENTS.md

## 公共基础设施约束

- 新增或修改 Docker Compose 基础设施前，必须先检查 `../shared-data-infra` 是否已经定义同类服务或 profile。
- 如果 `../shared-data-infra` 已定义 HDFS、Hive、Spark、YARN、Kafka、ZooKeeper、StarRocks、Prometheus、Grafana 等能力，不要在本工程重复新增；通过 external network、环境变量和项目级命名空间复用。
- HDFS、Hive Metastore、HDFS 初始化、HiveServer2、Spark、StarRocks FE/BE 均由 `../shared-data-infra` 提供。
- 本仓库的 `docker-compose.yml` 只保留 `benchmark-runner`。runner 通过 Docker socket 和可配置的共享基础设施 compose 文件控制共享服务生命周期。
- 只有 benchmark 隔离、性能对比或现有 runner 兼容性明确需要时，才允许保留 project-local 基础设施，并在 compose 或 README 中写明原因。
- 修改基础设施后，至少运行 `docker compose -f docker-compose.yml config`；涉及共享基础设施 compose 文件时，也要在 `../shared-data-infra` 运行相关 `docker compose ... config` 校验。

---
> Source: [w00125533/data-benchmark](https://github.com/w00125533/data-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
