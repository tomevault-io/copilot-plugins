## fatecat

> 当前根目录是 FateCat 企业级系统仓库真相源：结构采用 `apps`、`ai`、`domains`、`platform`、`infra`、`contracts`、`catalog`、`governance`、`shared`、`tools`、`docs`、`scripts`、`tests` canonical roots；运行、测试、导出和治理入口全部从企业根解析。

# AGENTS.md - FateCat Enterprise Repo

## 目录用途

当前根目录是 FateCat 企业级系统仓库真相源：结构采用 `apps`、`ai`、`domains`、`platform`、`infra`、`contracts`、`catalog`、`governance`、`shared`、`tools`、`docs`、`scripts`、`tests` canonical roots；运行、测试、导出和治理入口全部从企业根解析。

项目主旨：整理综合全部预测流派，首先完善中国传统主流和有效开源仓库，复用先于自写。

## 目录结构

```text
fatecat/
├── AGENTS.md
├── compose.yaml
├── DEBUG.md
├── README.md
├── REVIEW.md
├── SKILL.md
├── apps/
├── ai/
├── domains/
│   ├── fate-analysis/
│   │   └── services/fate-core/
│   │       ├── src/
│   │       └── tests/
│   └── experience-delivery/
│       └── services/fatecat-delivery/
│           ├── src/
│           ├── scripts/
│           └── tests/
├── platform/
├── infra/
│   └── docker/
│       ├── Dockerfile.delivery
│       └── entrypoint.delivery.sh
├── contracts/
├── catalog/
├── governance/
├── shared/
├── tools/
├── docs/
├── tests/
├── .github/
│   ├── AGENTS.md
│   └── workflows/
│       ├── acceptance.yml
│       └── container.yml
├── references/
│   ├── commands.md
│   ├── execution-playbook.md
│   └── troubleshooting.md
└── scripts/
│   ├── acceptance.sh
│   ├── check-structure.sh
│   ├── container-build.sh
│   ├── container-release.sh
│   ├── container-smoke.sh
│   ├── check-export-hygiene.sh
│   ├── check-source-hygiene.sh
│   ├── clean-runtime.sh
│   ├── delivery-smoke.sh
│   ├── export-runtime.sh
│   ├── live-bot-smoke.sh
│   ├── production-readiness.sh
│   ├── preflight.sh
│   └── vendor-health.sh
```

## 职责边界

- `SKILL.md`：标准 skill 入口说明。
- `compose.yaml`：本地和单机容器编排入口，只编排 delivery 容器与运行态 volume。
- `DEBUG.md`：当前调试证据、根因和回归验证记录；只承载已复现问题的诊断闭环。
- `REVIEW.md`：当前仓库审计结果与 release gate 结论；只记录证据、风险与交接，不承载业务源码。
- `apps/`：用户体验入口和渠道壳层。
- `ai/`：Agent、skill、Prompt、评估和 AI 监管相关入口。
- `domains/`：领域服务根；当前承载 `fate-core` 与 `fatecat-delivery` 两个生产候选服务源码、契约和测试入口。
- `platform/`：Golden Path、CI/CD、供应链和开发者平台能力。
- `infra/`：环境、容器、数据库、运行准入、安全和观测期望状态。
- `contracts/`：API、数据集、capability、profile、evidence 和策略契约。
- `catalog/`：组件发现、owner、生命周期和依赖关系。
- `governance/`：标准、流程、ADR、风险、门禁、baseline evidence、任务和迁移账本。
- `shared/`：真实复用后的薄共享库，不作为 common 垃圾桶。
- `tools/`：迁移工具、参考仓和供应链快照。
- `docs/`：人类文档入口，不替代机器契约和治理证据。
- `tests/`：仓库级结构、契约、导出和跨服务测试入口。
- `.github/`：GitHub Actions 远端验收配置；只调用仓库脚本，不保存业务代码或 secret。
- `references/`：长文档、阶段门禁、输入输出契约、迁移与排障材料；其中 `execution-playbook.md` 是统一执行顺序真相源。
- `scripts/`：本地可重复执行入口；其中 `preflight.sh` 是默认预检入口，`acceptance.sh` 是发布门禁入口，`check-structure.sh` 是企业结构门禁，`container-*.sh` 是容器构建、烟雾和发布入口。

## 依赖方向

- `apps/ai -> domains + contracts`
- `domains/experience-delivery -> domains/fate-analysis + contracts + infra`
- `domains/fate-analysis -> contracts + tools/reference-repos`
- `catalog -> domains + contracts + governance`
- `.github/workflows/acceptance.yml -> scripts/acceptance.sh`
- `.github/workflows/container.yml -> scripts/container-build.sh + scripts/container-smoke.sh`
- `scripts/* -> domains + contracts + infra + governance`
- 禁止新增旧路径 fallback；退役路径只允许出现在迁移账本、历史证据、负例测试和防回潮规则中。

---
> Source: [tradecatlabs/fatecat](https://github.com/tradecatlabs/fatecat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
