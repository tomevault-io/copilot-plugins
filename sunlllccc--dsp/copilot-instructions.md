## dsp

> mvn clean compile                          # 编译全部模块

# CLAUDE.md

## 1. 语言

Language=Chinese

## 2. 构建与运行

```bash
# 后端 (Maven, Java 21, Spring Boot 3.5.16)
cd dsp-parent
mvn clean compile                          # 编译全部模块
mvn clean compile -pl dsp-engine -am       # 编译单个模块及其依赖
mvn test -pl dsp-engine -Dtest=XmlEngineTest  # 运行单个测试类
mvn clean package -DskipTests              # 打包（跳过测试）

# 启动各服务
java -jar dsp-data-service/target/dsp-data-service-1.0.0.jar       # 端口 8080
java -jar dsp-offline-service/target/dsp-offline-service-1.0.0.jar # 端口 8081
java -jar dsp-admin-service/target/dsp-admin-service-1.0.0.jar     # 端口 8082

# 前端 (Vue 3 + Vite)
cd dsp-admin-web
npm install && npm run dev    # 开发服务器 :3000，代理 /dsp/admin→:8082，/dsp/offline→:8081
```

## 3. 模块依赖

包名 `com.sunlc.dsp`，`dsp-parent/pom.xml` 管理六个子模块：

```
dsp-common（无依赖）
dsp-core   → dsp-common
dsp-engine → dsp-common, dsp-core
dsp-data-service    → dsp-common, dsp-core, dsp-engine
dsp-offline-service → dsp-common, dsp-core, dsp-engine
dsp-admin-service   → dsp-common, dsp-core, dsp-engine
```

- `dsp-common` — 共享模型（ApiRequest/ApiResponse、ErrorCode、JwtUtil、DataQueryService 接口）
- `dsp-core` — CRUD 层（entity/mapper/service，MyBatis-Plus）
- `dsp-engine` — XML 执行引擎（详见 [docs/engine-architecture.md](docs/engine-architecture.md)）
- `dsp-data-service` — 数据查询 API，POST `/dsp/api/{transno}`
- `dsp-admin-service` — 管理后台 API，接口 CRUD / 审批 / 数据源 / 授权
- `dsp-offline-service` — 离线导出 XLSX/CSV/TXT

## 4. 技术栈

Java 21、Spring Boot 3.5.16、MyBatis-Plus 3.5.16（逻辑删除 `deleted` 字段）、Dynamic-DS 4.5.0 多数据源、DOM4J、Druid 1.2.28、JWT (jjwt 0.11.5)、Dubbo 3.2.11、EasyExcel 3.3.4、AgentScope Java 2.0.0-RC5
前端：Vue 3 + Element Plus + Pinia + Vite

## 5. 关键约定

- 数据源密码 AES 加密存储为 `ENC(base64)`，密钥在 `dsp.security.encrypt-key`
- JWT 密钥通过 `${DSP_JWT_SECRET:默认值}` 环境变量，生产必须更换
- 错误码：`ErrorCode` 枚举（0000=成功、4xxx=客户端、5xxx=服务端）
- 请求/响应统一 `ApiRequest<T>` / `ApiResponse<T>` 包装
- 数据库初始化脚本：`dsp-core/src/main/resources/db/init.sql`
- 修改引擎逻辑需充分手动验证

## 6. 开发规范

修改代码前先创建分支。详见 [CONTRIBUTING.md](CONTRIBUTING.md)（分支策略、提交规范、代码审查、版本管理）。

## 7. 行为准则

源自 Andrej Karpathy 的 LLM 编码最佳实践，偏向谨慎而非速度。

### 7.1 先想清楚

- 不确定就问，不要默默假设
- 存在多种理解时，列出选项让用户决定
- 如果有更简单的方案，说出来
- 不清楚就停，指出困惑点

### 7.2 简单优先

- 只写解决当前问题所需的最少代码
- 不加没要求的功能、抽象、"灵活性"
- 如果 200 行能缩到 50 行，重写

### 7.3 精准修改

- 只改必须改的，不"顺手"改周围代码、注释、格式
- 匹配现有风格，即使你觉得有更好的写法
- 你的变更导致的无用 import/变量，清理掉；但不动之前就存在的死代码
- 每一行变更都应能追溯到用户的需求

### 7.4 目标驱动

- 把任务转化为可验证的目标：「修复 bug」→「写一个能复现的测试，然后让它通过」
- 多步任务先列出计划，每步有验证检查点
- 编译通过、测试通过后再说「完成」

### 7.5 需求理解与行动 SOP

- 默认用户不是技术专家，用户描述可能不完整、不专业、带情绪；先用产品经理视角理解真实目标、使用场景、痛点和验收标准
- 不把用户的非专业表达当作最终技术方案；先复述对需求的理解，必要时把模糊表述转成可确认的产品目标
- 需求不清楚时，先问最少量的关键问题；如果能合理推断，说明假设并继续推进
- 动手前先给出简短 SOP：理解目标 → 拆分步骤 → 明确风险/依赖 → 说明验证方式
- 执行过程中保持耐心、稳定、少说教；把复杂技术翻译成用户能判断的结果、影响和取舍
- 发现用户方案可能不是最优时，先解释真实问题和更简单路径，再征得确认后调整
- 完成后用非技术视角说明做了什么、解决了什么、用户如何确认结果

## 8. 参考文档

- [引擎架构与执行流程](docs/engine-architecture.md) — XML 引擎的详细设计
- [XML DSL 配置参考](docs/xml-dsl-reference.md) — 接口配置语法手册
- [贡献指南](CONTRIBUTING.md) — 分支策略、提交规范、审查流程

---
> Source: [SunLLLccc/dsp](https://github.com/SunLLLccc/dsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
