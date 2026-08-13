## mercury-digital

> > 面向原创内容站长的自托管虚拟资源销售系统。

# Mercury — Vibe Coding Rules

> 面向原创内容站长的自托管虚拟资源销售系统。
> 资源商品交付类型：`file`（文件）/ `card`（卡密）/ `wiki`（在线知识库）  （积分支付购买）
> 现金充值业务：`points`（充值购买积分）/ `vip`（充值购买会员）  （微信/支付宝支付）
> 插件期功能（一期不做）：优惠券、签到、推广联盟、外链网盘、SaaS 插件市场。

---

## 1. 架构

**三层结构**

| 层 | 职责 |
|---|---|
| Kernel | 零业务逻辑基础设施：配置、JWT、网关、EventBus、WebSocket Hub、插件 KV 沙箱 |
| Modules | 核心业务：user / product / order / delivery / upload / wiki / ai / blockchain / settings |
| Plugins | 进程隔离外围扩展（HashiCorp go-plugin + gRPC），崩溃不影响主进程 |

**不可违反的约束**
- 使用 `uber-go/fx` 依赖注入，禁止全局 `init()` / 单例变量。
- 模块间禁止直接 `import`：异步用 `kernel/eventbus`，同步只读用内核共享接口（在 `kernel/auth` 中定义）。
- 多表写操作必须在 Service 层用 `db.Transaction`，杜绝扣款未发货。
- 所有用户可见文本（错误、label、推送）使用**简体中文**。
- 单文件不超过 300 行，按职责切割。

---

## 2. 目录结构

```
Mercury/
├── cmd/main.go                  # 唯一入口，仅做 fx.App 组装
├── kernel/
│   ├── auth/                    # JWT 生成/验证，Admin/Buyer 中间件，UserReader 共享接口
│   ├── config/                  # Bootstrap(YAML) + Runtime(DB 动态) 双层配置
│   ├── db/                      # GORM 初始化，自动建库，连接池
│   ├── redis/                   # Redis 客户端
│   ├── eventbus/                # Pub/Sub 事件总线（内存实现，可换 Redis）
│   ├── ws/                      # WebSocket Hub（订阅 EventBus，推送给在线用户）
│   ├── middleware/              # 限流、CORS、TraceID、审计日志、Panic恢复、Gzip
│   └── gateway/                 # 路由注册、插件注册表、插件 KV 代理
├── modules/
│   ├── user/                    # 注册/登录/资料/VIP状态/封禁；Admin：用户列表/手动授VIP
│   ├── product/                 # 资源商品CRUD+分类；PriceCalculator（查VIP专属积分价格）
│   ├── order/                   # 订单模块：微信/支付宝现金充值订单 (`recharge_orders`) + 积分消费兑换订单 (`points_orders`)
│   ├── delivery/                # 订阅 points_orders 的交付事件，编排卡密、文件或Wiki授权的派发
│   ├── upload/                  # 小文件上传 + 大文件三步分片（init/chunk/merge）
│   ├── wiki/                    # 空间/节点/阅读权限/DRM水印；Admin：空间和节点CRUD
│   ├── ai/                      # 向量化/RAG/SSE对话（可选，未配置则静默旁路）
│   ├── blockchain/              # 纯内核存证底座（定义 DDL 表、存证审计列表 API 及 Hook 事件分发，一期不开发链插件）
│   └── settings/                # system_settings 运行时配置读写；Admin：仪表盘/报表/审计日志
├── pkg/                         # 纯工具包（response/errcode/pagination/hash/validator）
├── sdk/proto/ + plugin_host/    # 插件 gRPC 契约
├── plugins/                     # 独立可执行插件进程
├── docs/                        # swag 生成的 Swagger
├── docker-compose.yml
└── go.mod
```

每个模块内部：`model/ → dto/ → repository/ → service/ → handler/`，依赖方向单向向下。

---

## 3. 数据库 Schema（19 张表，权威定义）

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE kernel_users (
    id             BIGSERIAL PRIMARY KEY,
    username       VARCHAR(64)  UNIQUE NOT NULL,
    email          VARCHAR(255) UNIQUE NOT NULL,
    password       VARCHAR(255) NOT NULL,
    role           VARCHAR(20)  NOT NULL DEFAULT 'buyer',  -- admin / buyer
    vip_level      INT          NOT NULL DEFAULT 0,
    vip_expire     TIMESTAMPTZ,
    points         BIGINT       NOT NULL DEFAULT 0,        -- 核心虚拟资产：积分余额
    is_banned      BOOLEAN      NOT NULL DEFAULT FALSE,
    wallet_address VARCHAR(128) UNIQUE,                    -- 预留 Web3 公钥
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE product_categories (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(64) NOT NULL,
    slug       VARCHAR(64) UNIQUE NOT NULL,
    sort_order INT         NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE products (
    id            BIGSERIAL PRIMARY KEY,
    category_id   BIGINT      REFERENCES product_categories(id) ON DELETE SET NULL,
    title         VARCHAR(255) NOT NULL,
    description   TEXT,
    price         BIGINT       NOT NULL,                   -- 价格单位：积分
    delivery_type VARCHAR(32)  NOT NULL,                   -- file/card/wiki (商品兑换只支持这三种类型)
    cover_url     VARCHAR(512),
    delivery_cfg  JSONB        NOT NULL DEFAULT '{}',
    -- delivery_cfg 规范：
    --   file:   { "file_id": "xxx" }
    --   card:   { "pool_id": "xxx" }  -- 仅支持单订单交付一张卡密，批量由插件扩展
    --   wiki:   { "space_id": 1, "days": 30 }  -- days=0为永久授权，days>0为到期天数
    validity_days INT          NOT NULL DEFAULT 0,         -- 0=永久，>0=购买后可再访问天数
    status        SMALLINT     NOT NULL DEFAULT 1,         -- 1=上架 0=下架
    is_consigned  BOOLEAN      NOT NULL DEFAULT FALSE,     -- 预留：代销标记
    is_original   BOOLEAN      NOT NULL DEFAULT FALSE,     -- 是否原创商品（勾选则需确认原创协议，承担侵权法律责任）
    original_stmt TEXT,                                    -- 原创责任声明承诺书文本快照（仅在 is_original=true 时有效）
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_products_fts ON products
    USING gin(to_tsvector('simple', title || ' ' || COALESCE(description,'')));

CREATE TABLE product_vip_prices (
    id         BIGSERIAL PRIMARY KEY,
    product_id BIGINT      NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    vip_level  INT         NOT NULL,
    price      BIGINT      NOT NULL,                       -- VIP专属价，单位：积分，0=免费
    UNIQUE(product_id, vip_level)
);

-- 现金充值订单（用户法币支付购买 积分 或 会员）
CREATE TABLE recharge_orders (
    id              BIGSERIAL PRIMARY KEY,
    order_no        VARCHAR(64)  UNIQUE NOT NULL,
    user_id         BIGINT       NOT NULL REFERENCES kernel_users(id),
    recharge_type   VARCHAR(20)  NOT NULL,                 -- points (买积分) / vip (买会员)
    package_cfg     JSONB        NOT NULL DEFAULT '{}',    -- 充值配置，如：{"points": 100} 或 {"vip_level": 1, "days": 30}
    amount          BIGINT       NOT NULL,                 -- 支付金额（单位：人民币分）
    status          VARCHAR(20)  NOT NULL DEFAULT 'pending', -- pending / paid / refunded
    paid_at         TIMESTAMPTZ,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- 积分兑换订单（用户消耗积分购买虚拟资源，纯本地数据库事务处理，无外部支付延迟）
CREATE TABLE points_orders (
    id              BIGSERIAL PRIMARY KEY,
    order_no        VARCHAR(64)  UNIQUE NOT NULL,
    buyer_id        BIGINT       NOT NULL REFERENCES kernel_users(id),
    product_id      BIGINT       NOT NULL REFERENCES products(id),
    points_paid     BIGINT       NOT NULL,                 -- 实际扣减的积分
    original_points BIGINT       NOT NULL DEFAULT 0,       -- 商品原价积分，对账用
    status          VARCHAR(20)  NOT NULL DEFAULT 'delivered', -- delivered / refunded
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE deliveries (
    id               BIGSERIAL PRIMARY KEY,
    order_no         VARCHAR(64)  UNIQUE NOT NULL REFERENCES points_orders(order_no) ON DELETE CASCADE,
    buyer_id         BIGINT       NOT NULL REFERENCES kernel_users(id),
    delivery_type    VARCHAR(32)  NOT NULL,
    token            VARCHAR(255),                         -- file 类型下载令牌
    payload          JSONB        NOT NULL DEFAULT '{}',
    token_expires_at TIMESTAMPTZ,
    created_at       TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE card_pools (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(128) NOT NULL,
    product_id BIGINT REFERENCES products(id) ON DELETE SET NULL,
    total      INT NOT NULL DEFAULT 0,
    used       INT NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE cards (
    id        BIGSERIAL PRIMARY KEY,
    pool_id   BIGINT      NOT NULL REFERENCES card_pools(id),
    card_data JSONB       NOT NULL,
    is_used   BOOLEAN     NOT NULL DEFAULT FALSE,
    order_no  VARCHAR(64) REFERENCES points_orders(order_no),
    used_at   TIMESTAMPTZ
);
CREATE INDEX idx_cards_pool_unused ON cards(pool_id) WHERE is_used = FALSE;

CREATE TABLE user_ledgers (
    id               BIGSERIAL PRIMARY KEY,
    user_id          BIGINT      NOT NULL REFERENCES kernel_users(id),
    amount           BIGINT      NOT NULL,                 -- 正增负减
    asset_type       VARCHAR(16) NOT NULL DEFAULT 'points', -- points (积分) / cash_cny (现金分)
    action_type      VARCHAR(32) NOT NULL,
    -- recharge (充值买积分) / buy_vip (充值买会员) / exchange (积分兑换商品) / refund (退款)
    related_order_no VARCHAR(64),
    description      VARCHAR(255),
    created_at       TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_user_ledgers_uid ON user_ledgers(user_id);

CREATE TABLE points_refunds (
    id          BIGSERIAL PRIMARY KEY,
    order_id    BIGINT      NOT NULL REFERENCES points_orders(id),
    user_id     BIGINT      NOT NULL REFERENCES kernel_users(id),
    reason      TEXT,
    points      BIGINT      NOT NULL,                       -- 退还积分数
    status      VARCHAR(32) NOT NULL DEFAULT 'pending',     -- pending / approved / rejected / completed
    operator_id BIGINT,
    note        TEXT,
    refunded_at TIMESTAMPTZ,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE wiki_spaces (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    description TEXT,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE wiki_nodes (
    id         BIGSERIAL PRIMARY KEY,
    space_id   BIGINT       NOT NULL REFERENCES wiki_spaces(id) ON DELETE CASCADE,
    parent_id  BIGINT,
    title      VARCHAR(255) NOT NULL,
    content    TEXT,                                       -- Markdown
    is_free    BOOLEAN      NOT NULL DEFAULT FALSE,
    sort_order INT          NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_wiki_nodes_fts ON wiki_nodes
    USING gin(to_tsvector('simple', title || ' ' || COALESCE(content,'')));

CREATE TABLE user_space_permissions (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT      NOT NULL,
    space_id   BIGINT      NOT NULL REFERENCES wiki_spaces(id) ON DELETE CASCADE,
    expires_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE(user_id, space_id)
);

-- ⚠️ 维度由 ai.embedding_provider 决定，建库后不可更改
-- local(bge-small-zh-v1.5)=vector(512)  remote(text-embedding-3-small)=vector(1536)
CREATE TABLE ai_embeddings (
    id            BIGSERIAL PRIMARY KEY,
    space_id      BIGINT NOT NULL REFERENCES wiki_spaces(id) ON DELETE CASCADE,
    node_id       BIGINT NOT NULL REFERENCES wiki_nodes(id) ON DELETE CASCADE,
    chunk_index   INT    NOT NULL,
    content_chunk TEXT   NOT NULL,
    provider      VARCHAR(16) NOT NULL DEFAULT 'local',
    embedding     vector(512) NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_ai_embeddings_hnsw ON ai_embeddings USING hnsw (embedding vector_cosine_ops);

CREATE TABLE blockchain_records (
    id           BIGSERIAL PRIMARY KEY,
    record_id    VARCHAR(64)  UNIQUE NOT NULL,
    record_type  VARCHAR(32)  NOT NULL,   -- PRODUCT_COPYRIGHT / RESOURCE_HASH
    data_hash    VARCHAR(128) NOT NULL,
    raw_data     JSONB        NOT NULL,
    tx_hash      VARCHAR(128),
    block_height BIGINT,
    status       VARCHAR(20)  NOT NULL DEFAULT 'pending',
    retry_count  INT          NOT NULL DEFAULT 0,
    confirmed_at TIMESTAMPTZ,
    created_at   TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

CREATE TABLE plugin_kv_store (
    plugin_id  VARCHAR(64)  NOT NULL,
    key        VARCHAR(128) NOT NULL,
    value      JSONB        NOT NULL,
    updated_at TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    PRIMARY KEY (plugin_id, key)
);

CREATE TABLE system_settings (
    key        VARCHAR(128) PRIMARY KEY,
    value      JSONB        NOT NULL,
    group_name VARCHAR(64)  NOT NULL,  -- ai/upload/cors/rate_limit/blockchain/saas
    label      VARCHAR(255) NOT NULL,
    is_secret  BOOLEAN      NOT NULL DEFAULT FALSE,
    updated_at TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_by BIGINT
);

CREATE TABLE notifications (
    id         BIGSERIAL PRIMARY KEY,
    user_id    BIGINT      NOT NULL REFERENCES kernel_users(id),
    notif_type VARCHAR(64) NOT NULL,   -- PURCHASE_SUCCESS/DOWNLOAD_READY/SYSTEM_ANNOUNCE
    title      VARCHAR(255),
    content    TEXT,
    is_read    BOOLEAN     NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE admin_audit_logs (
    id          BIGSERIAL PRIMARY KEY,
    admin_id    BIGINT      NOT NULL,
    action      VARCHAR(128) NOT NULL,
    resource    VARCHAR(64),
    resource_id VARCHAR(64),
    req_method  VARCHAR(10)  NOT NULL,
    req_path    VARCHAR(255) NOT NULL,
    req_body    JSONB,                                     -- 脱敏后（password/secret/key 替换为***）
    ip_address  VARCHAR(64),
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

---

## 4. 配置双层架构

| 层 | 存储 | 生效 | 内容 |
|---|---|---|---|
| Bootstrap | `config.yaml` / 环境变量 | 重启 | 仅 DB DSN、Redis、Port、JWT 密钥 |
| Runtime | `system_settings` 表 | 实时（Redis 缓存 TTL 5min） | 所有业务配置（AI/上传/CORS/限流/区块链/SaaS） |

Runtime 配置分组：`ai` / `upload` / `cors` / `rate_limit` / `blockchain` / `saas`  
`is_secret=true` 的字段（APIKey、PrivateKey）读取时脱敏返回。

---

## 5. 路由鉴权规则

```
🌐 公开路由   — 无 JWT：注册/登录/刷新Token/商品浏览/Wiki目录与免费章节/支付回调(充值订单)
👤 买家路由   — JWT(role=buyer|admin)：充值下单(积分/会员)/积分兑换资源/下载资源/已购Wiki/AI问答/通知/退款申请
🔑 管理员路由 — JWT(role=admin)：/api/v1/admin/** 全部接口
📝 审计       — 所有管理员写操作自动记录 admin_audit_logs
```

**封禁用户**：认证中间件检测 `is_banned=true` 时返回 `10008`，禁止通过任何鉴权。

---

## 6. 关键业务规则

**价格计算**：必须通过 `PriceCalculator` 计算 VIP 专属的**积分价格**：
1. 优先查 `product_vip_prices` 关系表是否有该商品的专属 VIP 价格。
2. 若没有，则读取系统配置（Runtime Settings）中配置的对应 VIP 等级的全局折扣率（如 VIP1 全场打 9 折，配置格式 `vip.level_1.discount = 90`）。
3. 若均未配置，则按商品原价 `products.price` 结算。

**充值交付（现金支付回调）**：当 `recharge_orders` 现金订单支付成功回调时，在单事务中：
1. 更新 `recharge_orders` 状态为已支付。
2. 写入现金账本流水 `user_ledgers(asset_type=cash_cny)`。
3. 执行充值交付：
   - 若充值类型为 `points`：给买家账户 `kernel_users.points` 增加充值的积分，并记录积分增发流水。
   - 若充值类型为 `vip`：读取 `package_cfg.days`，在原有到期时间基础上叠加天数（未激活从当前时间计算），更新 `kernel_users.vip_level` 和 `vip_expire`。

**积分兑换事务（本地资源购买）**：用户使用积分购买资源商品（file/card/wiki）时，在单个 `db.Transaction` 事务中进行：
1. 校验用户 `points` 余额是否充足。
2. 扣减用户积分 `kernel_users.points`。
3. 创建状态为已交付的积分兑换记录 `points_orders(status=delivered)`。
4. 写入积分变动流水 `user_ledgers(asset_type=points)`。
5. 执行具体交付逻辑并生成交付凭证 `deliveries`：
   - **卡密交付**：在单事务中通过 `SELECT ... FOR UPDATE` 锁住 `cards` 表中一条该卡池 `pool_id` 的未用卡密，标记为 `is_used=true`，绑定订单号 `order_no`。若库存不足，直接报错回滚整个事务。
   - **Wiki空间授权**：读取 `delivery_cfg.space_id` 和 `delivery_cfg.days`。若 `days=0`，在 `user_space_permissions` 中为用户开通该空间的永久授权（`expires_at=null`）；若 `days>0`，若无记录则从当前时间、若有未过期记录则在原有到期时间基础上累加天数。
   - **文件交付**：在 `deliveries` 写入交付快照，下载 Token 的 Redis 缓存有效期基于 `products.validity_days` 设定。
*由于不通过任何外部网络，纯数据库事务保证了 100% 的发货一致性。*

**支付路由**：仅用于现金充值。`is_consigned=false` → 调本地支付插件生成微信/支付宝付款码；`is_consigned=true` → 返回 SaaS 收银台 URL 进行中继（二期逻辑）。

**文件安全交付**：下载接口验证 `delivery_token`（存 Redis，TTL 由 `validity_days` 决定）后，本地存储用 `c.File()` 流式返回（隐藏磁盘路径），云存储返回带时限的预签名 URL（302 重定向）。

**AI 降级**：`ai.semantic_search_enabled=false` 或服务不可达或向量重建中，语义搜索静默降级为 GIN FTS，绝不 Panic 或暴露内部状态。

**Wiki 权限隔离**：pgvector 检索和 Wiki 内容接口必须强制过滤 `user_space_permissions`，无权空间数据绝不返回。

**Wiki DRM**：已购节点响应附加动态水印配置（用户名+模糊IP+时间戳），前端 SVG 全屏覆盖 + `MutationObserver` 防撕裂（10ms 内重建或清空内容）+ CSS/JS 复制拦截。

**区块链存证（仅内核底座，一期不实现具体链插件）**：
1. **插件化与底座设计**：Go 内核核心进程本身 **不包含任何具体的区块链客户端依赖**（如百度超级链或 FISCO BCOS 客户端 SDK），保持内核极度轻量与纯净。具体上链逻辑交由外围 gRPC 插件实现，支持未来随意更换链插件。
2. **触发与流转机制**：
   - 商品创建/编辑时，若勾选 `is_original=true` 并确认原创协议，内核向 `blockchain_records` 表写入一条 `status=pending` 的存证占位记录，并发布异步事件 `blockchain.record_created`。
   - 存证插件通过 Hook / EventBus 接收该事件，读取商品版权元数据并进行签名与哈希计算。
   - 插件自行调用链端 RPC 上链，成功后调内核网关接口将 `tx_hash` 和 `block_height` 反写回 `blockchain_records`，并将状态置为 `confirmed`。
3. **上链安全界限**：插件上链必须严格遵循 SF/T 0076—2020 规范，**只上链版权哈希与创作者元数据特征，绝对禁止上链任何订单金额、支付详情及买家个人隐私**。
4. **一期范围**：一期内核仅实现 `blockchain_records` 表结构、后台存证记录列表审计 API，以及对应的 Hook/事件分发底座。**具体上链的链插件在一期不予开发。**

---

## 7. API 协议约定

**响应信封**：`{ "code": 0, "message": "ok", "data": {}, "trace_id": "uuid" }`，所有接口统一，`code!=0` 时 `message` 为中文描述。

**错误码范围**：通用 `1xxxx`（参数/鉴权/限流/服务器错误）/ 业务 `2xxxx`（积分不足/卡密不足/Wiki无权限/下载过期等）/ AI `3xxxx`（未启用/调用失败）。新增错误码在 `pkg/errcode/errcode.go` 统一定义。

**分页**：入参 `?page=1&page_size=20`，出参 `{ "list":[], "total":0, "page":1, "page_size":20 }`，`page_size` 最大 100。

**JWT**：AccessToken 15min / RefreshToken 7天，`code=10003` 时前端自动无感续签，登出写 Redis 黑名单。

**WebSocket**：`ws://.../api/v1/ws?token=<AccessToken>`，Token 无效拒绝握手，单用户最多 5 连接，30s ping / 60s 无 pong 断开。

**限流**：Redis ZSet 滑动窗口，Redis 故障自动放行，触发返回 HTTP 429 + `Retry-After` 响应头，`/health` 和 `/readyz` 豁免。

**Swagger**：所有 Handler 必须写完整 swag 注释，挂载 `GET /swagger/index.html`。

---

## 8. 插件规范

- 进程隔离（HashiCorp go-plugin + gRPC），崩溃不影响主进程。
- **严禁插件直连数据库**，数据通过 KV 沙箱接口存入 `plugin_kv_store`。
- 插件通过 `Info()` 返回 JSON Schema，后台自动渲染配置表单，无需写前端。
- Hook 拦截点：`before_order_create`（可改价/拒单）/ `before_user_register`（可拒绝）/ `on_delivery`（代销插件拦截）。
- 本地安装：上传 `.zip` 包，主进程验证 `plugin.json` 后解压注册。

---

## 9. Docker 基础设施

| 服务 | 镜像 | 说明 |
|---|---|---|
| `postgres-db` | `pgvector/pgvector:16-pgvector` | 必须，预装 pgvector 扩展 |
| `redis-cache` | `redis:7-alpine` | 必须，带密码认证 |
| `grpc-text-embedding` | 自建（grpc_text 裁剪版） | **可选**，仅当 `ai.embedding_provider=local` 时启动 |
| `blockchain-node` | `fiscoorg/fiscobcos:v3.0` | **可选（一期默认不启动）**，FISCO BCOS v3 本地共识对等节点，负责与外部主网 P2P 互通同步 |
| `mercury-core` | 自建 | Go 后端主程序，依赖前三者健康检查通过后启动 |

容器内部通信域名：`postgres-db:5432` / `redis-cache:6379` / `grpc-text-embedding:50051` / `blockchain-node:30300`。  
`mercury-core` 挂载本地 `./uploads:/app/uploads` 作为文件存储目录。

---
> Source: [lip5201/Mercury-Digital](https://github.com/lip5201/Mercury-Digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
