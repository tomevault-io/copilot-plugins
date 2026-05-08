## frontbackapiguide

> AppConfig.apiBaseUrl 内部已拼接 “/api/v1/” 前缀，所有请求 path 必须以 “/” 开头，禁止再次写 /api/v1/**。

一、统一前置约定  
1. Base URL  
   AppConfig.apiBaseUrl 内部已拼接 “/api/v1/” 前缀，所有请求 path 必须以 “/” 开头，禁止再次写 /api/v1/**。  
   示例：`post('/ai-chat/sessions/create')` ✅           `post('/api/v1/ai-chat/sessions/create')` ❌  

2. HTTP 方法  
   • 除个别 GET-SSE（小说导入进度）外，所有接口一律使用 POST。  
   • 请求体、响应体统一 JSON；SSE 数据位于 data 字段内，亦是 JSON。  

3. 必备请求头  
   Authorization: Bearer {token}  
   X-User-Id: {userId}  
   Content-Type: application/json    （Multipart 上传除外）  
   SSE 额外头：  
   Accept: text/event-stream  
   Cache-Control: no-cache  
   Connection: keep-alive  

4. novelId 隔离  
   所有 AI 聊天 / 记忆模式 / 剧情推演及小说相关接口均需 novelId。前端省略将直接 4XX。旧版不带 novelId 方法已 @Deprecated。  

────────────────────────────────────────  
二、非流式（普通 HTTP）调用规范  
1. 调用方式  
   await apiClient.post('/路径', data: {…});  
   Options 可选；大部分接口已经在 ApiClient 二次封装，优先调用对应 repository 方法。  

2. 超时  
   connect 30s / send 30s / receive 5min（已在 ApiClient 配置）  

3. 错误处理  
   后端统一错误 JSON：`{ "code": -1, "message": "错误描述", "error": "可选详情" }`  
   ApiClient 已将 401 拦截并自动调用 AuthService.logout()。其它错误统一抛 ApiException(statusCode,msg)。  

────────────────────────────────────────  
三、SSE 流式请求规范  
1. 前端封装  
   使用 SseClient().streamEvents<T>(…)   或 ApiClient.postStream + _processStream。  

2. 通用请求头与 Options  
   见「统一请求头」，并在 Dio Options 中声明 responseType: ResponseType.stream。  

3. 事件格式（后端 ServerSentEvent）  
   id: {uuid}   ❘ 可选  
   event: {eventName}  
   data: {JSON 字符串或 [DONE]}  
   \n\n 分隔多条记录  

4. 事件名称白名单  
   chat-message             AI 聊天普通/流式块  
   chat-error               AI 聊天错误  
   chat-message-memory      记忆模式聊天  
   outline-chunk            剧情推演  
   message                  通用 AI  
   complete                 通用 AI 结束（data 为 {"data":"[DONE]"}）  

5. 结束判定  
   • 收到一条 data: [DONE]        → 正常结束  
   • 收到 event: complete         → 正常结束  
   • 服务器主动关闭连接          → onDone  
   • 本地 5 分钟未收包 → ApiClient 内置心跳超时会自动 addError & close  

6. 前端过滤范式（示例 chat_repository_impl）  
   .streamEvents(...).where((event)=>  
     event.sessionId == currentSession && event.content != 'heartbeat');  

────────────────────────────────────────  
四、路径与 DTO 命名  
1. 路径  
   POST /{模块}/{动作}        例：/novels/create  
   POST /{模块}/{资源}/{动作}  例：/ai-chat/messages/stream  
   SSE 路径保持同 POST 规则，仅响应类型不同。  

2. 请求 DTO/VO  
   • {Action}{Resource}Dto / Request / Response  
   • SessionCreateDto, ImportPreviewRequest …  
   • 流式请求的 DTO 放 body，不走 query。  

────────────────────────────────────────  
五、文件上传 & 导入  
1. Multipart FormData 字段  
   file                    文件  
   userId (备用)  
2. 三步导入  
   /upload-preview     → 返回 previewSessionId  
   /preview             → 返回章节解析预览  
   /confirm             → 返回 jobId  
   进度监听 GET /novels/import/{jobId}/status   SSE event: data = ImportStatus JSON  
3. 长连接心跳  
   ApiClient.connectToLongRunningSSE 已内建 15s 心跳日志；2min 静默 → 本地进度提醒；5min → 超时断线。  

────────────────────────────────────────  
六、特殊模块注意  
1. AI 聊天  
   • 创建会话      /ai-chat/sessions/create  
   • 流式发消息    /ai-chat/messages/stream   Body 必含 userId、novelId、sessionId、content  
   • metadata 可携带 aiConfig（详见 extractAIConfigFromMetadata）  
   • 事件过滤：status==streaming 可用于打字机效果，最终完整消息 id 不以 temp_chunk_ 开头。  

2. 记忆模式  
   • 路径加 -with-memory；event 名改为 chat-message-memory / chat-error-memory。  
   • 需要 memoryConfig 字段。  

3. 剧情推演  
   • POST /novels/{novelId}/next-outlines/generate-stream  
   • event: outline-chunk  
   • 如遇 code+message 错误 JSON，解析器需 throw ApiException。  

4. Universal AI（多功能 AI）  
   • 非流式 /ai/universal/request  
   • 流式 /ai/universal/stream   event: message / complete  
   • 预估费用 /ai/universal/estimate-cost  
   • 预览提示 /ai/universal/preview  

────────────────────────────────────────  
七、客户端实现要点（Dart 侧）  
1. ApiClient 已封装常用 CRUD；优先通过各 Repository，避免重复实现。  
2. 401 在拦截器中自动登出，无需额外判断。  
3. 日志等级：AppConfig.logLevel；生产默认 info，调试可设 warning 输出请求/响应体。  
4. 模型 & 配置缓存  
   ChatRepositoryImpl 内部维护 novelId→sessionId→UniversalAIRequest 缓存，注意同步 & 清理。  
5. 批量场景上传  
   统一使用 /novels/upsert-chapter-scenes-batch，数据结构符合 ChapterScenesDto。  

────────────────────────────────────────  
八、测试 Checklist（提交前自检）  
☐ 请求 path 无 “/api/v1” 重复  
☐ 必填头 Authorization / X-User-Id 已附加  
☐ POST body 为 JSON；SSE 请求 Accept:text/event-stream  
☐ novelId 已在 body 或 path 中提供  
☐ 错误码 & message 正确解析，401 能触发登出  
☐ SSE 解析：支持 event/data/id，多行合并，识别 [DONE]/complete  
☐ 心跳或空行已过滤，打字机流块保留  
☐ 长连接超时重连（最多 3 次）逻辑正常  
☐ 文件上传 Multipart/form-data，字段 file / userId  
☐ 日志在 debug 模式下可输出请求体 & 响应体  





后端约束

一、通用约束  

1. 路径前缀  
   所有 Controller 均挂载在 “/api/v1/**”，切勿在内部拼接重复前缀。  

2. 返回类型  
   • 非流式：Mono<T> / Flux<T> ↔ HTTP 200|201。  
   • 流式  ：Flux<ServerSentEvent<…>> ↔ Content-Type text/event-stream。  

3. DTO & 命名  
   • 输入 DTO：SessionCreateDto / ImportPreviewRequest / PaginatedScenesRequestDto …  
   • 输出 DTO：NovelWithScenesDto / ChaptersForPreloadDto …  
   • 统一放在 web.dto 包；禁止 Controller 直接暴露实体 Entity。  

4. 参数校验  
   • 使用 jakarta.validation @Valid，并在 DTO 字段加 @NotBlank @NotNull。  
   • Controller 若自行拼 Map<String,String>，在进入 Service 前必须手动判空并抛 ResponseStatusException 400。  

5. 身份认证  
   • 使用 @CurrentUser 解析出 userId；如为空必须回退表单 userId；仍为空则返回 401。  
   • 鉴权逻辑统一在 Service 层做二次校验（session 属主、novel 属主等）。  
   • jwt配置 SecurityConfig需要增加新端点

6. novelId 隔离  
   • AIChatService / NovelService 等新接口必须带 novelId 版本，旧方法保留 @Deprecated 标记。  
   • Controller 在调用旧 Service 时，先 getSession(userId, novelId, sessionId) 校验归属。  

7. 错误响应规范  
   ```json
   { "code": -1, "message": "错误描述", "error": "可选堆栈/详情" }
   ```  
   • Controller 捕获异常后统一封装，避免直接返回 500 HTML。  
   • SSE 端点 onErrorResume 时应推送 chat-error / outline-error 之类事件，或者 data: {"code":-1,"message":"xxx"}。  

────────────────────────────────────────  
二、SSE 端点实现要求  

1. 标准包装  
   ```java
   ServerSentEvent.<T>builder()
       .id(UUID.randomUUID().toString())
       .event("chat-message")   // 见事件白名单
       .data(payload)           // payload 为对象，框架自动 JSON 序列化
       .retry(Duration.ofSeconds(10))
       .build();
   ```  

2. 事件名称  
   chat-message | chat-error | chat-message-memory | outline-chunk | message | complete  
   其它请先在前后端约定后再扩展。  

3. 结束规则  
   • 业务方最后 concat 一个 complete / data:[DONE] 信号，或正常 close。  
   • 服务器不得无限保持空闲长连接；无事件 2-3min 应考虑心跳注释 “:heartbeat”。  

4. 流速控制  
   使用 `.delayElements(Duration.ofMillis(50))` 或下游 buffer，避免单秒百条刷屏。  

5. 订阅日志  
   `doOnSubscribe` 记录连接；`doOnCancel` / `doOnError` 记录关闭与异常，方便排障。  

────────────────────────────────────────  
三、模块专项  

1. AI 聊天 (AIChatController)  
   • createSession / getSession / listSessions / update / delete / count 全量支持 novelId。  
   • streamMessage() 必须先 extractAIConfigFromMetadata() → UniversalAIRequestDto；如无配置降级旧接口。  
   • 错误时返回 chat-error 事件，data 为 AIChatMessage（role=system, status=ERROR）。  

2. 记忆模式  
   • routes 加后缀 -with-memory；事件名 chat-message-memory / chat-error-memory。  
   • ChatMemoryConfigDto 转 domain，Service 侧负责窗口剪裁。  

3. Novel 管理 (NovelController)  
   • get-with-paginated-scenes 等分页接口必须校验 chaptersLimit 1-10。  
   • load-more-scenes direction 仅允许 up/down/center；非法值 400。  
   • 细粒度增删改（add-act-fine / delete-scene-fine 等）只处理局部，无需回整本 DTO，前端自动拉取最新结构。  
   • /import 流程：upload-preview → preview → confirm；每步严格校验必要字段并返回友好错误。  

4. Next-Outline (剧情推演)  
   • generate-stream / regenerate-option 均推 outline-chunk。  
   • Service 内部 chunk.size 建议 ≤ 5KB；过大前端解析慢。  

5. Universal AI  
   • /stream 结尾必须 concat 完成事件 `event:complete data:{"data":"[DONE]"}`。  
   • /estimate-cost 返回 {success, estimatedCost, errorMessage}，不可抛异常给前端。  

────────────────────────────────────────  
四、性能 & 稳定性  

1. I/O 超时  
   • WebClient/DQL 调 OpenAI 等第三方应限 2min；大任务另行异步处理并用 SSE 推进度。  

2. 压力保护  
   • 单 userId 并发流连接 ≤ 10；可在 Service 层做计数。  
   • 若超额返回 429 JSON 并在 SSE 推送 error 事件。  

3. 日志  
   • slf4j 级别：info 记录业务流程 & 关键ID；debug 打开 JSON 细节；error 打印堆栈。  
   • 不得在生产输出完整 prompt / apiKey。  

────────────────────────────────────────  
五、代码质量守则  

1. Controller 只做参数检查 + 日志 + 调 Service；禁止业务逻辑堆叠。  
2. Service 返回 Mono.error 时务必带语义化 message，前端直接展示。  
3. DTO 层禁止 Lombok @Data；使用 @Getter/@Setter 或 record，避免 JSON 循环引用。  
4. 所有 Mono/Flux 链路结尾必须 `onErrorResume` 友好处理，不能把 Reactor 异常原样抛给客户。  
5. 不得在 SSE 控制器里使用 `share()` 导致多次订阅；一个请求一个冷流或 Service 级共享 hot 流。  

────────────────────────────────────────  
六、提交前检查清单（后端）  
☐ 路径不含重复 /api/v1  
☐ DTO 字段 @NotBlank 检验通过，全局异常处理返回统一结构  
☐ novelId 校验正确，跨用户/跨小说数据隔离  
☐ SSE 事件名符合白名单，结尾发送 complete 或 [DONE]  
☐ 日志不输出敏感信息(apiKey, prompt)  
☐ 新增接口在 Controller + Service + DTO 均写单元测试  

---
> Source: [Deng-m1/MaliangAINovalWriter](https://github.com/Deng-m1/MaliangAINovalWriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
