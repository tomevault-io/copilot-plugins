## ai-link-net

> 2. **代码减少重复和耦合**：只要有同样的代码出现两次，及时解耦，减少代码。你可以走完功能后，统一查看一遍，有哪些重复代码，比较简单的过程可以定义一个内部函数，来减少代码；必要的时候创造类来提供固定方法来解耦代码，比如自定义的 Printer、Logger等。

# GUIDANCE
当前阶段：快速开发阶段 aln@v0.1

## 请严格遵循代码简洁之道:

1. **严格按照 OOP 原则**

2. **代码减少重复和耦合**：只要有同样的代码出现两次，及时解耦，减少代码。你可以走完功能后，统一查看一遍，有哪些重复代码，比较简单的过程可以定义一个内部函数，来减少代码；必要的时候创造类来提供固定方法来解耦代码，比如自定义的 Printer、Logger等。

3. **auto review 减少代码行数**：添加新功能以后,跑通了以后,要 review 代码,减少代码行数

4. **提高代码类型安全**：尽量所有类都用类型注释,BaseModel，同时 要少用 dict[str,Any] 这种形式不安全,代码可读性差，尽量少用

5. **代码要优美,像诗歌一样可读**：命名要斟酌,必要的时候先和 用户讨论清楚变量、函数、类的命名再开始开发

6. **deps**：所有的 import 放到头部，不要在函数内 import，import 的顺序要标准 内置库 > 引入库 > 自建内容，同一级按字母顺序

7. **docstring**：非核心函数的 docstring 只写一行就行

8. **# NOTE :**重点关注代码中的  **# NOTE** ，这是我写的提醒事项，你要严格遵守这些注意点，**不允许你改动 # NOTE** ：不允许你自己写 #NOTE！,只有用户自己来写NOTE，你负责遵守

9. **#TODO:** 在实现负责功能的时候优先进行代码抽象，用 pass 代替实现，并加上 **#TODO**，同时和用户讨论各个类、方法的功能边界、相互调用关系等

10. **批判性思维**：当我要你添加、修改新功能的时候，你先自行分析判断我的想法的合理性、可行性，然后给我建议并和我讨论确定。保证我们的思路最优，方案细化可行的时候，再去执行。

## 请严格遵循如下开发规则:

1. **当用户让你自行提交变更的时候**，你要主动查看即将被 add 的变更，和同级别文件进行对照，对照变更后的风格是否一致，修改的代码有没有重复冗余。

2. **当用户让你自行提交变更的时候**，你要主动全量检查 docs，看看修改代码的同时，有没有同步更新对应内容的文档。如果有设计原则性的冲突，给我建议并和我讨论，确定是要修改代码还是修改文档后再执行。

3. **当初在快速开发阶段的时候，你的代码无需做向后兼容**：这时候你要考虑的更多是扩展性设计，为未来补全留空间，也可以用 pass 来类、函数、方法占位，并留 TODO 来记录

## 1. 仓库主线与优先级

- `fp` 协议层已拆分为独立仓库 [foundation-protocol](https://github.com/FoundationAgents/foundation-protocol)，通过 git 依赖引入，包名仍为 `fp`。
- 当前主线代码在 `aln/app` + `aln/cli`，这是优先优化区。
- `aln/cli` 和 web 层还在追赶最新抽象，改动时要优先跟随 `aln/app` 的稳定接口。

## 2. 分层边界（必须遵守）

- `fp` 层（外部依赖 `foundation-protocol`）：协议核心与领域模型，不做应用级持久化与进程编排。修改 fp 层代码需到 foundation-protocol 仓库操作。
- `app` 层（`aln/app`）：运行时、API、配置读写、对外服务。
- `cli` 层（`aln/cli`）：命令编排和展示，不承载核心业务逻辑。
- `web` 层：界面与调用聚合，不重写后端协议规则。

依赖方向：

- 允许：`app -> fp`、`cli -> app/fp`、`web -> app API`
- 不允许：`fp -> app`

## 3. 核心类与职责边界

### 3.1 协议核心（外部依赖 `foundation-protocol`，包名 `fp`）

- `FPAddress`（`fp/core/base.py`）：
  - 负责地址格式与解析（`host_uid:entity_uid`）。
  - 不负责拓扑管理和网络通信。
- `Message` + `MessageKind`（`fp/message.py`）：
  - 负责业务消息语义（kind/payload/metadata/ack）。
  - 不负责路由和传输。
- `Mail`（`fp/mail.py`）：
  - 负责传输信封（sender/recipient/message/signature）和签名/加解密封装。
  - 不负责决定投递路径。
- `Entity`（`fp/entity.py`）：
  - 负责实体身份、好友关系、消息收发入口、mail 校验。
  - 不负责网络层拓扑和持久化策略。
- `Host`（`fp/host.py`）：
  - 负责 Host 内/跨 Host 的路由、实体注册更新删除、拓扑关系（parent/child）。
  - `save()` 仅留协议层接口，持久化细节放应用层覆盖。
- `Cryptor` / `Ed25519Cryptor`（`fp/core/cryptor.py`）：
  - 负责加解密/签名抽象。
  - 算法切换通过实现新 Cryptor，而不是在 Mail/Entity 里塞分支。
- 以上源码修改需到 [foundation-protocol](https://github.com/FoundationAgents/foundation-protocol) 仓库操作。

### 3.2 应用层（`aln/app`）

- `HostServer`（`aln/app/service/host_server.py`）：
  - 应该承接 `Host` 的应用层能力（WSS 转发、进程生命周期、UI 进程控制）。
  - 目前很多方法是 `pass`，新增功能优先补齐这里，不要反向污染 `fp.Host`。
- `HostClient`（`aln/app/service/host_client.py`）：
  - 对外 Host API 访问入口，封装 HTTP 请求、错误处理、响应反序列化。
  - CLI/其他上层需要访问远端 Host 时优先复用它。
- `HostConfig`（`aln/app/service/host_config.py`）：
  - 本地配置文件读写与锁管理的唯一入口。
  - 不要在其他地方直接复制一套 json 文件读写逻辑。
- `StandardResponse[T]`（`aln/app/schemas/response.py`）：
  - API 响应统一信封。
  - 新接口优先复用，而不是返回散乱 dict。

### 3.3 命令层（`aln/cli`）

- CLI 命令函数只做参数编排和调用，不做深业务逻辑。
- 使用 `get_host_client` 注入 HostClient。
- 使用 `get_cli_printer` 注入输出器。
- 使用 `cli_exception_wrapper` 做统一异常处理。

## 4. 改哪里时要一起改哪里（联动清单）

1. 改 `Host`/`Entity` 公共方法签名（在 foundation-protocol 仓库）：
   - 同步检查：`aln/app/api/v1/*`、`aln/app/misc/provider.py`、`test/app/*`。
2. 改 `Message`/`Mail` 字段或序列化逻辑（在 foundation-protocol 仓库）：
   - 同步检查：`aln/app/service/host_server.py`、`aln/app/handlers/*`、`test/app/*`。
3. 改 API 路径：
   - 同步检查：`aln/app/endpoint.py`、`aln/app/api/*` 路由、`aln/app/service/host_client.py`、web 前端、CLI 对应命令。
4. 改 schema（如 `HealthResponse`、`EntityUpdateRequest`、`HostUpdateRequest`）：
   - 同步检查：`aln/app/api/*` response_model、`HostClient` 反序列化、CLI 输出、web 类型定义。
5. 改配置结构（`HostConfig`）：
   - 同步检查：`aln/cli/host.py` 全部子命令和任何读取 host url/pid 的逻辑。
6. 改 discover/well-known 结构（在 foundation-protocol 仓库）：
   - 同步检查：`aln/app/service/host_client.py`、`children/parent` API、web host 类型定义。

## 5. 复用地图（不要重复造轮子）

- fp 协议层工具（在 foundation-protocol 仓库维护，通过 `fp.*` 导入）：
  - 路径工具：`fp.utils.path` — `get_xxx_path()`、`ensure_xxx_dir()`
  - 消息 ID：`fp.utils.common._new_message_id`
  - keypair：`fp.utils.common.generate_keypair`
  - mailbox 默认路径：`fp.utils.common.default_mailbox_path`
  - 存储管理：`fp.utils.storage`
- API 异常包装：
  - `aln/app/misc/exception_handler.py::exception_wrapper`
- URL 校验：
  - `aln/app/misc/validation.py::normalize_parent_url`
- CLI 注入器：
  - `aln/cli/misc/wrappers.py` 的 `get_host_client`、`get_cli_printer`、`cli_exception_wrapper`
- 进程工具：
  - `aln/cli/misc/process.py` 的 `is_pid_alive`、`stop_pid`
- 统一 endpoint 常量：
  - `aln/app/endpoint.py::ENDPOINT`

## 6. 代码习惯总结（从仓库现状提炼）

- 倾向 `Pydantic BaseModel + Field + validator` 做类型与数据约束。
- 倾向 `from __future__ import annotations`。
- 文档注释风格偏“核心方法写清边界，非核心保持一行简洁 docstring”。
- 对外接口倾向 schema 化；避免裸 dict 蔓延。
- 已多处写明“应用层负责持久化，协议层保留抽象接口”。

## 7. 新功能落地工作流（按你的要求执行）

1. 先判断需求合理性与可行性，必要时先讨论后再改。
2. 实现优先放在正确层级，不跨层硬塞逻辑。
3. 功能跑通后做一轮“减行 review”：
   - 去重
   - 合并重复分支
   - 提升命名可读性
4. 回看类型安全：
   - 能用明确模型就不用 `dict[str, Any]`
   - 输入输出尽量用 `BaseModel` 承载

## 8. 当前仓库状态提醒

- 当前存在“接口重构中”的迹象（部分 `pass`、部分调用名与核心类未完全对齐）。
- 结论：新增功能应以 `fp` 核心模型和 `app` schema 为中心收敛，不要在 `cli/web` 里先发散新协议。

---
> Source: [FoundationAgents/ai-link-net](https://github.com/FoundationAgents/ai-link-net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
