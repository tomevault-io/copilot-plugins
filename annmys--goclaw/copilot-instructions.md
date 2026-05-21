## goclaw

> 这些规则适用于 `D:\goclaw` 本地代码仓库。

﻿# GoClaw 代码仓库规则

这些规则适用于 `D:\goclaw` 本地代码仓库。

## 线上安全

- 默认保护当前线上功能，不要为了升级破坏现有业务。
- 不要破坏当前 `Kimi Code` Provider、租户、权限、skill 路由、定时任务和数据库迁移。
- 合并 GitHub 上游更新前，必须先评估对本地自定义修改的影响。
- 即使是低风险更新，也要验证服务运行状态。

## 代码修改

- 修改前先查看现有实现方式，保持项目原有风格。
- 优先小范围、可验证的补丁，不做不必要的大重构。
- Go、TypeScript、React、SQL migration、Docker 配置都要遵守现有结构。
- 未经明确要求，不要修改公开的 skill slug、任务名称、租户标识、模型名称和 Provider 名称。

## 测试

- 后端修改要运行对应 Go package 测试。
- 前端修改要尽量运行 build、typecheck 或对应测试。
- 数据库修改要验证 migration 能执行，`schema_migrations` 不能是 dirty。
- Docker 或部署修改后，要验证容器健康状态和相关页面/接口。

## git

- 修改前后检查 `git status`。
- 不要 amend commit，除非用户明确要求。
- 不要使用破坏性 git 命令，除非用户明确授权。

## GitNexus

- GitNexus 跑通后，优先用它做代码图谱和影响分析。
- 如果 GitNexus 索引失败，要记录环境、命令、退出码和临时解决方案。

# 自动提炼规则
- ...` 的条目，才会被自动提炼到对应 `AGENTS.md`，普通日志不会自动写入规则文件。

---
> Source: [Annmys/goclaw](https://github.com/Annmys/goclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
