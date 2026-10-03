## doumcp

> - **类型**：LSPosed 模块（libxposed API 102），注入抖音进程内启动 MCP 服务（Ktor Streamable HTTP，端口 19320，Bearer 鉴权）

# DouMCP（抖M）- 抖音 MCP 模块

## 项目概览

- **类型**：LSPosed 模块（libxposed API 102），注入抖音进程内启动 MCP 服务（Ktor Streamable HTTP，端口 19320，Bearer 鉴权）
- **宿主**：抖音 `com.ss.android.ugc.aweme`
- **结构**：单模块 Gradle 工程，Kotlin，`app/` 下分包

```
app/src/main/java/io/github/yfishyon/doumcp/
  ModuleMainKt.kt        入口：进程判断、hook 安装、热重载
  McpServerHost.kt       MCP 服务宿主（Ktor + 各功能包工具注册）
  ModulePrefs.kt         用户配置（端口/密钥/开关，FastKV 原生 API）
  SettingsInjector.kt    抖音设置页注入"抖M设置"
  core/                  共享设施：DexKitSupport（桥懒加载+缓存+并发搜索）/ ResolvedCache /
                         Reflect / ModLog（libxposed 日志）/ DouToastHelper / McpToolExt（toolCall 输出校验）
  features/              按功能分包，每个功能包内固定 tool / bridge / resolver 三层：
    account/             账号列表/切换/用户资料
    im/                  会话/消息/表情/输入状态（最大的功能包）
    video/               视频详情/用户作品
    comment/             评论读取/发布
    search/              综合搜索
    system/              系统工具（getCurrentTime 等）
```

## 构建与测试

```bash
./gradlew assembleDebug    # debug 包测试（别用 --no-daemon）
```

- 构建出 debug 包后安装到设备，等 10 秒热重载生效
- 日志走 libxposed 通道（`ModLog`），logcat 过滤 DouMCP（tag 前缀是 LSPosedLogDaemon）
- 用户提示：MCP 服务启动后、DexKit 首次建索引完成后都会弹抖音风格 toast（DouToastHelper）
- 热重载不生效（日志无"热重载完成"）→ 强停抖音重新启动，等 25 秒

## 硬性约束

1. **代码和注释里绝不写混淆的类名/方法名/字段名**（未混淆固定名可以）。注释一律陈述式，不写"实测/调试/临时/看完删"这类字样
2. **禁止猜任何键值**：优先逆向（jadx / smali 注解 / 模型 toString）→ 运行时确认 → 必要时临时 dump（看完即删）
3. **确定键值后去掉防御性编程**：不层层兜底、不候选轮询、不写死白名单（运行时过滤，有多少给多少）
4. 改代码用 Edit/Write；python 只允许真正的批量替换
5. 测试一律 debug 包；群聊参数用 conversationId
6. **需登录的功能入口先查 `AccountBridge.getCurrentUserId`**，未登录返回 `{"ok":false,"error":"未登录"}`
7. **MCP 工具返回统一 `{"ok":true/false,...}` JSON 结构**——`toolCall` 会校验（非法 JSON / 缺 ok 字段会被拦截）；豁免：`ping`（裸 pong，连通性检查）
8. 调试代码用完即删；诊断性 Log.i 不留，只留错误与关键节点

## 日志

- 只用 `core/ModLog`（libxposed 通道，入口 `onModuleLoaded` 时 `ModLog.bind(this)`）
- `i`：启动/热重载等关键节点；`w`：降级路径；`e`：异常与定位失败。禁止 `android.util.Log`

## 存储

- **FastKV 原生 API**（`io.github.billywei01:fastkv`）：`FastKV.Builder(context, name).build()` + putXxx/getXxx，完全不用 SharedPreferences 接口
- 配置用 `ModulePrefs`，DexKit 定位缓存用 `ResolvedCache`；key 不要改（老数据读不到）
- DexKit 缓存 key 携带宿主版本：`ResolvedCache.versionedKey(versionCode, feature)`

## 新增一个功能（feat）的步骤

1. 建包 `features/<feat>/{tool,bridge,resolver}`
2. `resolver/` 写定位器：matcher 用 DexKit 全等字符串锚点，走 `DexKitSupport.resolveCached`（缓存样板已收敛），候选类搜索用 `searchCandidatesParallel`（并发）
3. `bridge/` 写能力桥：入口做登录检查，返回统一 JSON（带 ok 字段）；字段值运行时过滤，不写白名单
4. `tool/` 写注册：`internal fun Server.registerXxxTools(hostClassLoader)`，参数用 `stringArg/intArg`，包在 `withContext(Dispatchers.IO) + toolCall { }` 里
5. `McpServerHost.registerTools` 的 registrations 列表挂上注册函数
6. 描述文本写清楚参数含义与取值来源（方便 MCP 客户端使用）

## 定位模式（DexKit）

- 字符串/类锚点**必须全等匹配**（`StringMatchType.Equals`），锚点抄完整
- 定位结果两级缓存：内存 + FastKV（按宿主版本隔离）；缓存全命中不建 DexKit 索引
- 候选类搜索用 `DexKitSupport.searchCandidatesParallel`（并发，不要写串行循环）
- 抽象方法不能 hook；Retrofit 接口找实现类或调用方
- 调宿主挂起方法：Proxy 宿主 classloader 的 Continuation + 宿主 EmptyCoroutineContext；挂起标记用 `is Enum<*> && name == "COROUTINE_SUSPENDED"` 判定

## 服务端可见标识

以下固定字符串随请求上送，可识别模块来源（有意保留）：
- 视频详情/子评论请求的 channel 参数、评论发布的 enter_from 均为 `doumcp`

## 数据库工具（features/database）

- 工具：listDatabases / listTables / queryDatabase / executeDatabaseStatement（camelCase 统一）
- 输出为扁平 KV 文本（「N row(s):」+「列=值, 列=值」）；写语句模式「Statement executed.」
- 加密库（encrypted_ 前缀）用宿主 WCDB 兼容层打开，密钥由文件名里的 uid 推导（任意账号的库都能解）
- 连接缓存（密钥派生耗时，懒建）；宿主删库/查询失败时弃连接重建
- IM 库时间戳是毫秒（13 位）；个别 ext 字段是秒（10 位），换算前先看位数
- **不要在宿主首次 WCDB 使用前触发其类初始化**（静态初始化依赖宿主上下文，抢跑会崩宿主 IM 启动）；也**不要在 hook 回调栈内同步 unhook**（框架等待回调退出会自死锁）

## 热重载

- onHotReloading 停服务（给排空时间）+ 停自有线程（executor shutdown，契约要求）
- 端口预检失败重试 3 次再放弃（旧服务排空竞态）

---
> Source: [yfishyon/doumcp](https://github.com/yfishyon/doumcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
