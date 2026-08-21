## raffle

> 本文件是当前项目的 Agent 地图，只放项目定位、模块边界、常用命令和关键规范入口。

# AGENTS.md

本文件是当前项目的 Agent 地图，只放项目定位、模块边界、常用命令和关键规范入口。

## Agent skills

### Issue tracker

Issues 以 GitHub Issues 形式跟踪（仓库 github.com/yyyac/raffle），通过 `gh` CLI 操作。详见 `docs/agents/issue-tracker.md`。

### Triage labels

使用五个默认 triage 标签：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。详见 `docs/agents/triage-labels.md`。

### Domain docs

单上下文布局：仓库根 `CONTEXT.md` + `docs/adr/`。详见 `docs/agents/domain.md`。


## 项目概述

本项目是一个大营销业务系统，核心围绕抽奖活动配置、库存扣减、用户参与、策略计算、规则过滤、中奖记录、奖品发放、行为返利、积分账户、积分支付兑换、消息发送与任务补偿等能力展开，支撑从营销活动创建到用户抽奖、发奖和后续补偿的完整业务链路。

## 仓库结构

- `big-market-yyya-api`：对外 API 契约层，包含接口定义、请求/响应 DTO，以及统一返回对象 `Response<T>`。
- `big-market-yyya-trigger`：输入适配层，包含 HTTP Controller、RabbitMQ Listener、定时 Job，只负责接收请求或消息并调用领域服务。
- `big-market-yyya-domain`：核心领域层，按业务领域拆包，包含领域对象、领域服务、领域事件和 Repository 接口。
  - `activity`：活动领域，负责活动配置、SKU 库存、活动账户额度、活动下单、支付兑换、参与抽奖等逻辑。
  - `strategy`：抽奖策略领域，负责策略装配、奖品概率表、规则链、规则树、库存扣减和抽奖结果计算。
  - `award`：奖品领域，负责中奖记录、奖品发放、发奖任务和积分奖品等发放实现。
  - `rebate`：行为返利领域，负责用户行为返利、返利订单、返利消息事件和任务补偿。
  - `credit`：积分领域，负责积分账户、积分流水、积分交易调整和积分成功事件。
  - `task`：任务领域，负责消息发送任务、补偿任务和任务状态流转。
- `big-market-yyya-infrastructure`：基础设施层，包含 Repository 实现、MyBatis DAO、PO、Redis 服务、MQ 事件发布和外部网关适配。
- `big-market-yyya-types`：通用类型层，包含通用常量、枚举、异常、注解和事件基类。
- `big-market-yyya-app`：应用启动层，包含 Spring Boot 启动类、配置类、AOP、`application-*.yml`、MyBatis XML、日志配置和测试用例。
- `docs`：项目文档、开发规范、子 Agent 协作说明、Docker Compose 环境编排、应用启停脚本和 MySQL 初始化 SQL。
- `data`：本地运行产生的数据或日志目录，提交前注意不要混入无关运行日志。

## 构建、测试与本地运行

- `mvn clean package`：从根目录构建全部模块并打包启动 Jar。
- `mvn -pl big-market-yyya-app -am test -DskipTests=false`：运行 app 模块及依赖模块测试。
- `mvn -pl big-market-yyya-app -am spring-boot:run -Dspring-boot.run.profiles=dev`：以 dev 配置本地启动应用。

## 架构红线

- `trigger` 只调用 `api` 契约和 `domain` 领域服务，不直接引用 `infrastructure` 的 DAO、PO、Redis、MQ。
- `domain` 只依赖领域对象、领域 Repository 接口和 `types`，不引用 Controller、DAO、PO、Redis、MQ、MyBatis 或 Spring Web 等技术实现。
- `infrastructure` 实现 `domain` 定义的 Repository 接口，负责持久化、缓存、消息发布和外部网关适配。
- `app` 负责应用启动、配置装配、AOP、资源文件和测试，不承载核心业务规则。
- `types` 只放通用常量、枚举、异常、注解和事件基类，不能反向依赖业务模块。

更细的开发红线、高风险变更清单和范围扩大处理见 `docs/development-guide.md`。

## 测试指南

当前测试主要使用 JUnit 与 Spring Boot Test，测试类以 `*Test.java` 结尾，集中在 `big-market-yyya-app/src/test/java/com/yyya/test`。新增领域规则、Repository 事务、Controller 流程或 MQ/Job 补偿逻辑时，应补充相邻包下的测试。

测试职责分工、无法自动化验证时的说明要求，以及提交前验证规则见 `docs/development-guide.md`。

---
> Source: [yyyac/raffle](https://github.com/yyyac/raffle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
