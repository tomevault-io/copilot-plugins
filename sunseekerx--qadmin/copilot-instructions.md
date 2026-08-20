## qadmin

> 通用规则在全局 ~/.claude/CLAUDE.md（对话自动加载；项目内镜像 CLAUDE_GLOBAL.md 与其逐字一致，全局更新后同步刷新镜像）。本文件只放 qadmin 项目专属规则，「项目须提供 X」的落地位置在「项目概述」小节；与全局冲突时以本文件为准。

# 版本: <260703.7>

通用规则在全局 ~/.claude/CLAUDE.md（对话自动加载；项目内镜像 CLAUDE_GLOBAL.md 与其逐字一致，全局更新后同步刷新镜像）。本文件只放 qadmin 项目专属规则，「项目须提供 X」的落地位置在「项目概述」小节；与全局冲突时以本文件为准。

# qadmin 项目规则

## 项目概述

qadmin 是开源通用管理后台底座。部署即全新：每次发布清空 MySQL + Redis + ClickHouse，从 init.sql + clickhouse_ddl.sql 重建。因此不存在增量迁移——所有表结构、种子数据直接维护在 server_data/shared/mysql/init.sql，新功能的建表/改表/种子一律改 init.sql 本体，禁止另建迁移文件。

- server/：管理后台 API。NestJS 10 + TypeORM + MySQL + ioredis + Bull
- server_admin_ui/：管理后台前端。Vue 3 + Vite + Element Plus + Pinia
- server_go/：管理后台 API（Go 版基座，与 NestJS 同构：system/login/monitor）。Gin + GORM + MySQL + go-redis + viper
- server_rust/：管理后台 API（Rust 版基座，与 NestJS 同构：system/login/monitor）。Axum 0.8 + SeaORM + MySQL + fred + Tokio
- servers_ref/server_py/：管理后台 API（Python 版基座，与 NestJS 同构）。FastAPI + SQLAlchemy + MySQL + redis-py
- servers_ref/server_php/：管理后台 API（PHP 版，全量对齐 NestJS）。Webman(Workerman) + illuminate/database + illuminate/redis(predis)
- servers_ref/server_java/：管理后台 API（Java 版，全量对齐 NestJS）。Spring Boot + MyBatis-Plus + MySQL + Redis
- servers_ref/server_kotlin/：管理后台 API（Kotlin 版，部署上接管全部 cron）。Spring Boot + MyBatis-Plus + MySQL + Redis
- servers_ref/server_csharp/：管理后台 API（C# 版，对齐 NestJS）。.NET 10 + SqlSugar + MySQL + Redis

- 目录分层即优先级：主力 server/、同构基座 server_go/ server_rust/ 留在仓库根；其余降权参考实现（Python/PHP/Java/Kotlin/C#）统一收纳在 servers_ref/ 下。这 5 个后端按 cwd-相对引用共享数据目录时用 ../../server_data/...（比根级后端多一层 ../）

- 后端结构：src/modules/（框架功能：system/login/monitor/cache）、src/modules_biz/（业务模块，如 nav）、src/common/、src/shared/
- Go 基座结构：internal/base/（system/login/monitor 基座）、internal/app/（装配）、pkg/、config/、cmd/server（入口）
- Rust 基座结构：crates/{core,infra,middleware}（基础设施）、crates/{mod_system,mod_monitor}（功能模块）、crates/app（装配入口）
- PHP 结构：app/common/（契约+PHP8 attributes）、app/middleware/（反射读 attribute 的守卫链）、app/process/（worker 注册 + cron 调度常驻进程）、app/modules/（system/login/monitor + 额外模块）、app/biz/nav、config/
- 多套后端（NestJS/Go/Rust/Python/PHP）同构，共库部署。Go/Rust 与 server_php 一样全量对齐 NestJS（含 tenant/sms/mail/social/notify/i18n/sensitive_word/file/api_log/gen/nav）
- [人工决策-2026-06-19 14:02:41] Go/Rust 由「剥离业务的纯基座（只保 system/login/monitor）」改为全量对齐 NestJS 所有功能，推翻 [人工决策-2026-06-17 22:41:57] 中关于 Go/Rust 为纯基座的定位（该决策的目录分层部分仍有效）
- Redis key 文件：server/src/modules/sys_redis_keys.ts（NestJS）；servers_ref/server_php/app/common/RedisKeys.php（PHP）
- 时区配置：后端环境变量 BIZ_TIMEZONE（默认 Asia/Shanghai），前端环境变量 VITE_BIZ_TIMEZONE（默认 Asia/Shanghai）。各端读取位置——NestJS: server/src/common/env.ts env.bizTimezone + formatBizTime()；Go: server_go/config/config.go app.biz_timezone + pkg/utils/time.go BizLoc/FormatBizTime()/ParseBizTime()；Rust: server_rust/crates/core/src/config.rs app.biz_timezone + pretty_log.rs init_biz_timezone()；前端: server_admin_ui/src/shared/libs/env.js env.bizTimezone(←VITE_BIZ_TIMEZONE)、时间库 luxon 在 shared/libs/lib.js(utcToBiz 做 UTC→业务时区显示)。业务时区「日期范围」→ UTC 转换（纯日期查整天、datetime 精确、begin/end 各自可选、右闭、DST 安全）：NestJS server/src/common/env.ts bizDateRangeWhere()（返 TypeORM where 操作符），Go server_go/pkg/utils/time.go BizBeginToUtc()/BizEndExclusiveToUtc()，Rust server_rust/crates/core/src/time_utils.rs biz_begin_to_utc()/biz_end_exclusive_to_utc()。部署时前后端统一配置同一个 IANA 时区值
- 配置与时间库入口：业务配置单一入口 NestJS server/src/common/env.ts（process.env 只在此读，dotenv 引导在 main.ts、进程身份 SHARD_ID 用 NODE_APP_INSTANCE/pid 属例外）、前端 server_admin_ui/src/shared/libs/env.js（import.meta.env 只在此读）；时间库统一 luxon，server 从 src/common/libs.ts 导入 DateTime/luxonFrom、前端从 shared/libs/lib.js 导入 DateTime 与 formatDateTime/formatDateTimeShort/formatDate/formatUtc 助手，禁止直接 import luxon
- [人工决策-2026-06-14 04:46:46] server_php 共存与架构决策：运行时 Webman(Workerman) 常驻 worker；所有后端共享同一套 Redis key（admin:/sys:/captcha: 用于 session/缓存/在线，qadmin: 用于协同层 worker 注册+cron 锁）；定时任务全端从 Bull 改分布式锁调度(方案 A，锁 key qadmin:lock:cron:{jobId}:{unixMinute} NX PX90s，分钟粒度，NestJS 已去 Bull 用法)；代码生成器产出 Vue+PHP(Webman) 模板
- [人工决策-2026-06-28 19:17:54] Redis key 前缀统一为 qadmin: 全前缀，推翻 [人工决策-2026-06-14 04:46:46] 中"session/缓存/在线用裸 admin:/sys:/captcha:"的部分（该决策的 cron 锁/worker 注册/方案A/代码生成器部分仍有效）。原文字与 NestJS 真值源代码(sys_redis_keys.ts 早已全 qadmin: 前缀 + snake_case 段名，如 qadmin:admin:token/qadmin:admin:nick_name/qadmin:sys:config/qadmin:sys:dict/qadmin:captcha:{uuid})矛盾，导致 java/php/csharp(裸前缀阵营)与 NestJS/go/rust/kotlin(qadmin:阵营)共库时登录态互不可见。已统一 java/php/csharp 对齐 NestJS（含散落硬编码点：online strip/cache 监控分组 pattern/config·dict SCAN pattern；注意 clearCacheAll 须只清 qadmin:admin/captcha/sys 排除 workers/lock 运行态）
- 开发环境用 docker-compose 启动 MySQL、Redis 和 ClickHouse，见 README.md
- 各后端 dev 启动命令：NestJS cd server && npm run dev（:3000）；Go cd server_go && make dev（:9711，需 air）或 make run（编译后运行）；Rust cd server_rust && just dev（:9811，需 cargo-watch）或 just run（单次运行）。Rust APP_ENV 默认 development 无需显式指定
- 包管理分栈：后端 server/（及 llysc/pi 等同构后端）用 bun 装依赖（bun add/bun remove，不是 pnpm）；前端 server_admin_ui/ 用 pnpm。需安装/卸载的依赖照旧告诉用户、由用户自行执行
- 前端 CSS 优先用原子 CSS（Tailwind）：能用原子类（flex、mb-4、text-sm、任意值类 text-[var(--xxx)] 等）就用原子类直接写在模板上，不新增 <style scoped>。仅当原子类表达不了（复杂选择器、::v-deep、关键帧动画、:deep() 穿透组件库样式）才写 scss；改现有 scss 文件时跟随现有风格
- 定时任务显式时区（通用规则「定时任务」的本项目落地）：NestJS @Cron({ timeZone })、Spring @Scheduled(zone)；多端同一 cron 抢锁细节见「多后端架构与协同契约」
- [人工决策-2026-06-29 12:48:41] 发布交付方式分两类：能产原生二进制的两个后端直接交付单文件二进制——server_go（make build → tmp/server）、server_rust（just build，即 cargo build --release -p qadmin_app → target/release/qadmin_app），拷到目标机零运行时直接跑；其余不产二进制的后端一律 Docker 镜像发布——server(NestJS/Node)、servers_ref/server_py(Python)、servers_ref/server_php(PHP)、servers_ref/server_java(Java/JVM)、servers_ref/server_kotlin(Kotlin/JVM)、servers_ref/server_csharp(C#/.NET，虽可 Native AOT/self-contained 出二进制但当前 csproj 未配，统一归 Docker)。前端 server_admin_ui 是构建出的静态资源，走 nginx 托管（可打进 nginx 镜像，见 nginx.conf.example），不在「二进制 vs Docker」分类内
- [人工决策-2026-06-29 13:34:52] 6 个 Docker 后端的生产 + 本地开发镜像化：① 每个后端生产 Dockerfile（多阶段/非 root/HEALTHCHECK；NestJS 那份是既有的生产完备实现保持不动，kotlin 那份重写并把 TZ=Asia/Shanghai 修正为 UTC，py/php/java/csharp 新建）；② 本地开发用独立 Dockerfile.dev（容器内热重载，本机零工具链），不动生产 Dockerfile；③ Go/Rust 不做 dev 镜像（dev 仍本机跑）；④ dev 编排各后端自管：6 个后端各自目录有 docker-compose.dev.yml + Makefile（make dev-up/dev-rebuild 等，各自独立 project name 互不影响），NestJS 也可不走容器直接本机 npm run dev，不进根 package.json、保留「根 package.json 只 4 条」原状。完整文档见 DEPLOY.md「本地开发（Docker）」「参考实现后端镜像」两节
- [人工决策-2026-07-03 13:26:21] 全部后端均支持 Jenkins + Docker 部署：补齐 servers_ref 下 c/cpp/ruby/swift 的生产 Dockerfile + Jenkinsfile + deploy.sh（契约与其余参考后端一致：ENV=production/production_hk、MODE=source/image 双模式、蓝绿切换 + 健康检查 + 失败回滚、配置 .env.${ENV} 常驻目标机 --env-file 注入、sshPublisher 带 failOnError:true + continueOnError:false、source 分支带 ERR trap 自清），扩展 [人工决策-2026-06-29 12:48:41] 的 Docker 阵营与 [人工决策-2026-06-29 13:34:52] 的镜像化范围（go/rust 产二进制的部分仍有效；c/cpp/swift 亦可产原生二进制作为补充交付）。生产端口沿用 dev 端口 +1 规律：ruby 10012 / swift 10112 / cpp 10212 / c 10312，前缀 /api-ruby /api-swift /api-cpp /api-c；这 4 端健康检查用「有 HTTP 响应即存活」判定（无根路由，404 也算活）；ruby 生产另需 SECRET_KEY_BASE（Rails 要求）。demo 演示流水线与前端切换器仍为 8 后端，未纳入这 4 端
- 跨机部署流水线(Jenkins Publish over SSH，仓库所有 Jenkinsfile：server/·server_admin_ui/·server_go/·server_rust/·servers_ref/*/·server_data/demo_deploy/Jenkinsfile.demo，均已置)必须让远程失败真正中止：sshPublisher 默认 failOnError=false，传输/连接/远端 exec 失败只打 ERROR 不抛、流水线假成功继续往下跑（曾致 reset 连接被拒仍进后续构建、部署阶段远端没执行也当成功）——凡跨机/远程步骤(含新增 Jenkinsfile 或 sshPublisher 调用)一律显式带 failOnError:true + continueOnError:false。另：目标机构建档(MODE=source)时各 deploy.sh 在蓝绿 rollback trap 安装之前就解压 + docker build，失败会把 _src/ 与 src_bundle.tar.gz 残留在部署目录，故 9 个 deploy.sh 的 source 分支各带一个临时 ERR trap 自清后再退（判例见各 deploy.sh source 分支注释）
- [人工决策-2026-06-20 17:26:57] 引入 ClickHouse（仅分析用，弱依赖：缺失不阻塞启动、不参与业务决策，沿用通用块「ClickHouse 随时可丢」），为 proxy_pool 代理切换/请求日志统计服务，推翻原「本项目无 ClickHouse」；CH 建表人工执行，DDL 在 server_data/shared/clickhouse/clickhouse_ddl.sql（库名 dev=qadmin_dev / prod=qadmin），代码标记见 server/src/common/env.ts、server_go/config/config.go、server_rust/crates/core/src/config.rs
- 共用数据目录 server_data/shared/：各后端共用的种子(mysql/init.sql)、上传(uploads)、dev 日志(logs/<backend>)；开发命令统一在仓库根 package.json，只 4 条：npm run setup(起 MySQL+Redis+ClickHouse + 灌种子 + 建 CH 表，出来即完整可开发态)、npm run reset(重灌 MySQL + 清 Redis + 重建 CH 表，三者全清干净)、npm run up/down(起停，up 含 CH)；仅灌种子/仅清 Redis/仅建或清 CH 表(chc 加 reset 参数清空)等 niche 操作走裸 docker compose ... run --rm <mysqlc|redisc|chc> 不占别名。dev 默认起 CH 是为完整可开发态，不违背 CH「运行时弱依赖」(app 仍不 require CH、CH 挂了自愈降级 503)。npm 仅作入口，编排逻辑(等就绪/重建库/灌库/清 Redis/建 CH 表)全在容器内跑(server_data/dev_tools/：复用 mysql/redis/clickhouse 官方镜像当客户端，接入 dev 栈网络 qadmin_dev_default)，宿主机只需 Docker、不装任何客户端，三端零差异。详见 server/README.md

## Entity 与 init.sql 对齐（DB_SYNCHRONIZE 双轨）

- dev DB_SYNCHRONIZE=true、prod false：init.sql 必须与实体逐字对齐，否则 dev 每次启动 churn。时间列写 datetime DEFAULT NULL（禁 datetime(6) ... CURRENT_TIMESTAMP）；索引要 dev 保留须在实体加 @Index。新表/列直接改 init.sql + reset（通用块「启动时禁止 DDL」只约束生产）
- init.sql ↔ 实体对齐检查清单（dev churn 三大高频来源，看到 [TypeORM] columns changed in "xxx" 按此逐列排查）：① 可空性——实体属性不带 nullable: true 即 NOT NULL，init.sql 对应列必须写 NOT NULL（@Column({ default: '' }) 等价 NOT NULL DEFAULT ''，漏写 NOT NULL 必 churn）；② 整型宽度——@PrimaryGeneratedColumn 及无显式 type 的数字列 TypeORM 默认 int，init.sql 勿写成 bigint；③ column comment——TypeORM 在 MySQL 下会比对列 comment，须与实体 comment: 逐字一致（含括号内说明、中文全角括号 （）、斜杠两侧空格），实体里的行内 // 注释不是 comment，别误抄进 init.sql
- init.sql 种子数据的时间列一律写 UTC 值（通用块「时区」的本项目补充）

## 多后端架构与协同契约

- NestJS(server/) 是唯一真值源：业务先在 NestJS 落地，再同构移植。改动先改 NestJS，Go/Rust 紧跟同构，其余后端为参考实现、不保证功能完整
- [人工决策-2026-06-17 22:41:57] 降权参考实现按目录分层：server_py/server_php/server_java/server_kotlin/server_csharp 移入 servers_ref/，仓库根只留 server(主力) + server_go/server_rust(同构基座) + server_admin_ui + server_data。下沉一层后这 5 个后端对 server_data 的 cwd-相对引用统一改为 ../../server_data/...；以 server_data/...(仓库根执行) 或 ${ROOT}/...(绝对根) 形态的引用不变
- 开发优先级：NestJS 主力 > Go/Rust 同构后端（全量对齐 NestJS，见 [人工决策-2026-06-19 14:02:41]）> Python/PHP/Java/Kotlin/C# 参考实现（已物理下沉到 servers_ref/，与目录分层对应）
- [人工决策-2026-06-19 16:18:05] 所有后端共享同一个 MySQL + Redis，可共享登录态，各后端唯一的区别是功能实现进度不同。表命名、Redis key 命名、模块结构对齐同源项目（D:\code\ssx\llysc、D:\code\pt\pt_mj\pi_node_manager）：系统表用 sys_ 前缀（sys_user/sys_role/sys_dept/sys_menu/sys_post/sys_config/sys_dict_type/sys_dict_data/sys_notice/sys_ui_pref），监控表用 mon_ 前缀（mon_oper_log/mon_logininfor）。当前 qadmin NestJS 的无前缀表名（user/role/dept）是早期遗留，待统一改为 sys_ 前缀对齐同源
- 协同层全后端统一（qadmin: 命名空间）：① Worker 注册到 qadmin:workers Hash（workerId 带语言标识 + 心跳，僵尸惰性清理），monitor 在线列表跨语言聚合；② Cron 每分钟 tick 抢 qadmin:lock:cron:{jobId}:{unixMinute}(SET NX PX90s)，抢到才执行、本端无该 handler 则跳过让给他端（详见 [人工决策-2026-06-14 04:46:46]）
- 协同层例外：C# 的 job 锁非分钟粒度（仅 C# 间互斥、不与他端互斥），慎同时部署多个 C#；Kotlin 部署上专职接管全部 cron
- 跨后端契约必须各后端逐字一致：redis key 命名、qadmin:workers/cron 锁结构、i18n 错误码 + catalog（见下）。改契约同步所有后端；key 文件指针见上「Redis key 文件」

## i18n 与错误码契约 [人工决策-2026-06-14 18:09:40]

- 路线：不引 nestjs-i18n，扩展自研 i18n（server/src/common/i18n/，决策理由见 error_code.ts 顶部注释）
- 错误响应信封 { code, msg, errCode? }：code 粗粒度状态码（前端判成功/重登的开关，200/401/500，语义不可改）；msg 按请求语言渲染；errCode 新增的语言无关语义码（客户端 key off 它做自定义文案/分支），仅 bizError() 抛出的异常带
- 错误码统一注册：业务异常一律 bizError('CODE')（注册表 error_code.ts），禁止 throw new ApiException('硬编码中文')；新错误码加一行注册 + 在 locales.ts 各语言包补同名 catalog key
- 校验失败：全局 ValidationPipe 的 exceptionFactory 按 class-validator 约束名自动翻译（validation.* catalog，DTO 无需逐个加 message）；DTO 显式填 catalog key 可精细覆盖
- 语言开关（env）：默认按 Accept-Language 协商；I18N_ENABLED=false 或设 I18N_FORCE_LANG 锁定单语言（只服务中国大陆设 false 即全程中文）；兜底语言 I18N_DEFAULT_LANG（默认 zh-CN）
- errCode + catalog 是跨后端契约：语义码与 key 命名各后端逐字一致，任一后端服务同一请求翻出的文案一致；前端 @/libs/error_code 可扩展为 errCode→文案映射做客户端翻译

## 分页器（挡位配置 + 按位置记忆）[人工决策-2026-06-11 16:00:39]

- 挡位全集 10/20/30/50/100/200/300/500/1000；实际显示哪些挡位由系统配置 key pagination_page_sizes_admin 控制（系统配置页可改），配置缺失/非法时前端用全集兜底
- 保存语义 [人工决策-2026-06-11 17:31:00]：本项目纯管理后台无用户侧，偏好只用单一表 sys_ui_pref（登录用户存 DB，跨设备生效）；不做匿名 localStorage 路径，不做用户自定义挡位
- 按位置记忆：每个分页位置一个唯一 storage key（<域>:<位置> 风格，按 views 路径取名，如 system:user、monitor:job、system:dict_data），用户切换挡位后记住该位置的选择，下次进入用记忆值
- 机制 src/utils/page_pref.js：登录后 permission.js 拉一次（挡位配置 + 偏好映射）；页面 pageSize 初始化必须包 pagePref.initPageSize('<key>', 默认值)；统一分页组件 <pagination> 传 storage-key 自动保存；裸 el-pagination 绑 :page-sizes="pagePref.pageSizes" + @size-change 调 pagePref.savePageSize
- 新增表格分页必须接入：initPageSize + storage-key（或 el-pagination 三件套）；禁止再写死 :page-sizes="[...]"
- 切换挡位后必须重查：已用 @pagination 回调的天然满足（size-change 只做保存）；只有 @current-change 的要在 size-change 中回第 1 页并重拉

---
> Source: [SunSeekerX/qadmin](https://github.com/SunSeekerX/qadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
