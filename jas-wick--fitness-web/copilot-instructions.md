## fitness-web

> FitHub 是一个多用户在线健身管理平台。后端 Spring Boot 3 提供 REST API，前端 Vue 3 + Vite 通过 HTTP 访问。

# FitHub：基于 Spring Boot + Vue 的智能健身管理平台

## 1. 项目概述

FitHub 是一个多用户在线健身管理平台。后端 Spring Boot 3 提供 REST API，前端 Vue 3 + Vite 通过 HTTP 访问。

核心功能：

- **训练打卡**：基础/进阶双模式打卡，月度日历、连续打卡统计、趋势分析
- **饮食记录**：餐次分组、营养素构成可视化
- **身体数据**：体重/体脂/BMI 记录与趋势
- **动作教学**：42 个标准动作，分类 + 搜索 + 图解（SVG 剪影）
- **数据分析**：训练次数/时长/热量趋势、训练类型占比
- **社区**：发帖、点赞、收藏、转发、评论
- **AI 健身助手**：基于真实数据生成训练计划 / 饮食分析 / 数据复盘（智谱 GLM）

## 2. 技术栈

- **后端**：Java 21 / Spring Boot 3.3 / Spring MVC / MyBatis-Plus 3.5.7 / MySQL 8 / Redis 7（本机用 Memurai 兼容服务）/ Spring Security 6 + JWT(jjwt 0.12) / Lombok / Validation / springdoc-openapi
- **前端**：Vue 3 + TypeScript + Vite + Element Plus + Pinia + Axios + ECharts + @element-plus/icons-vue + marked / dompurify
- **构建**：Maven（后端）/ npm（前端）

## 3. 项目结构

```
fitness-web/                # FitHub 项目根目录
├── CLAUDE.md                # 本文件：项目开发规范
├── README.md
├── docker-compose.yml       # MySQL8 + Redis7（标准环境）
├── docs/                    # architecture.md / database.md（设计文档）
├── server/                  # Spring Boot 后端（com.fitness）
│   ├── scripts/             # svg-only.mjs（动作 SVG 生成）
│   └── src/main/java/com/fitness/
│       ├── common/{result,exception,constant,util}
│       ├── config/          # DataInitializer / Cors / Security 等
│       ├── security/        # JwtAuthFilter、SecurityConfig、TokenService
│       ├── controller/ service/impl/ mapper/ entity/ dto/ vo/
│       └── ai/              # AiService + provider/
└── client/                  # Vue3 + Vite 前端
    ├── public/exercises/    # 42 个动作 SVG 静态资源（随仓库提交）
    ├── scripts/             # screenshot.mjs / repro-comment.mjs（QA 工具）
    └── src/
        ├── api/             # Axios 请求封装 + 各模块 API
        ├── components/      # 可复用组件（StatCard/ChartCard/PageHeader 等）
        ├── composables/     # useECharts/useList/useDate/useConfirm/useViewport/useAuthAvatar
        ├── constants/       # 共享常量（训练类型/餐次/健身目标等）
        ├── layouts/         # MainLayout（深色侧边栏 + Header）
        ├── router/ stores/ types/ utils/
        ├── styles/          # tokens.css（设计 Token）+ index.css（全局样式）
        └── views/           # 11 个页面 + NotFoundView
```

## 4. 核心开发原则

- **分层单向依赖**：`controller → service(impl) → mapper`；`entity` / `dto` / `vo` 三者分离，禁止 controller 直接暴露 entity
- **DRY**：优先复用现有组件 / composables / 常量 / 工具，避免复制粘贴（页面内重复的 `today()`/`formatTime()`/分页逻辑已被收敛）
- **不改后端为视觉目的**：UI/UX 改动不得修改后端接口、数据结构或伪造数据
- **保留现有契约**：API、Pinia、Axios 封装、路由的既有约定不得破坏
- **真实数据优先**：图表/统计基于真实 API 数据；无数据时用空态，绝不虚构

## 5. 后端开发规范

- **统一返回**：`Result<T>{code, message, data}`；分页 `PageResult<T>{records, total, page, size}`；全局异常由 `GlobalExceptionHandler` 处理
- **参数校验**：jakarta validation 注解 + `@Validated`；密码 BCrypt
- **数据库**：无物理外键（阿里规范），逻辑外键 + 索引；逻辑删除 `deleted` + `@TableLogic`；主键 `BIGINT UNSIGNED AUTO_INCREMENT`；表前缀 `t_`；库名 `fitness`（10 张表，见 `docs/database.md`）
- **基包** `com.fitness`；后端 artifactId `fitness-server`
- **日志**：SLF4J + Logback（分环境），见 `logback-spring.xml`

## 6. 前端开发规范

- **共享常量**：一律放 `client/src/constants/index.ts`。注意：`TRAINING_TYPES`（短名：胸/肩/背…，写入训练集数据）与 `EXERCISE_BODY_PARTS`（全名：胸部/背部…，动作库筛选）**语义不同、值不同，不得合并或改值**
- **复用 composables**：
  - `useECharts`：图表 init/resize/dispose 封装（配合 ChartCard 使用）
  - `useList`：通用分页列表状态
  - `useDate`：`today()` / `formatTime()` 等日期工具
  - `useConfirm`：删除确认（ElMessageBox）
  - `useViewport`：断点监听（<1024 为移动端）
  - `useAuthAvatar`：受保护头像 → token 拉取 blob URL 展示
- **类型**：DTO 类型统一在 `client/src/types/index.ts`
- **图标**：用 `@element-plus/icons-vue`（main.ts 已全量注册），**不用 emoji** 当图标
- **请求**：统一走 `client/src/api/request.ts`（拦截器已注入 token、解包 Result、401 自动刷新重放）

## 7. UI / UX 规范

- **设计 Token**：`client/src/styles/tokens.css` 是全部视觉规范的唯一来源（主色运动橙 `#F97316`、深色侧边栏 `#111827`、页面背景 `#F8FAFC`、圆角/阴影/间距/字号），**必须在 `element-plus/dist/index.css` 之后引入**（main.ts 中顺序即正确）；全局基础样式在 `client/src/styles/index.css`
- **图表**：一律通过 `ChartCard` 组件（内部 useECharts，暴露 `setOption/clear/resize`），**不手写 echarts init/dispose**；图表要有标题/单位/tooltip/空态
- **统一组件**：`StatCard`（数值卡）、`PageHeader`（页头）、`EmptyState`（空态）、`FormDialog`（弹窗壳）、`ImageWithFallback`（图片兜底）、`MacroProgress`（营养素条）、`TrainingCalendar`（打卡日历）
- **状态反馈**：异步操作要有 Loading / Empty / Error / Success / 删除确认 / 表单校验；不裸显 `undefined`/`null`
- **响应式**：`useViewport` 断点；<1024px 侧边栏收进 `el-drawer`；用栅格与 `min-width:0` 防溢出
- **动作图片**：`/exercises/{slug}.svg` 为前端静态资源（`client/public/exercises/`，随仓库提交），由 `DataInitializer` 播种 `image_url`；重新生成用 `server/scripts/svg-only.mjs`

## 8. API 规范

- RESTful；统一 `Result<T>` 包体；分页查询统一 `page` / `size` 参数
- 前端拦截器已解包 `Result`，页面直接拿 `data`
- 公开端点：`POST /api/auth/register|login|refresh`、`GET /api/exercise/**`、`GET /api/stats/site`、swagger（`/swagger-ui/**`、`/v3/api-docs/**`）
- 管理端：`/api/admin/**` 需要 `ADMIN` 角色
- 其余端点需登录

## 9. Redis 使用原则

统一使用 `StringRedisTemplate`（**不使用 `RedisTemplate` / 不再有 `RedisConfig`**）。实际使用的 key（以 `Constants.java` 为准）：

| Key | 用途 | TTL |
|---|---|---|
| `fitness:refresh:{userId}` | refresh token | 7 天 |
| `fitness:blacklist:{jti}` | 登出吊销 | 与 token 有效期一致 |
| `fitness:exercise:hot` / `fitness:post:hot` | 热度榜 | 1800s（续期） |
| `fitness:post:views` | 帖子浏览量 | 无 |
| `fitness:stats:site` | 站点统计缓存 | 有界 |
| `fitness:login:fail:{username}` | 登录失败限流 | 900s |
| `fitness:training:stats:{userId}` / `fitness:training:streak:{userId}` | 训练统计缓存 | 30 天 |

## 10. Security

- **认证**：JWT access token（2 小时）+ refresh token（7 天，存 Redis）；登出写黑名单吊销
- **登录限流**：`fitness:login:fail:{username}` 超过 5 次锁 15 分钟
- **角色**：`/api/admin/**` 仅 ADMIN；前端为单角色用户端
- **CORS**：来源白名单（`application-dev.yml` 允许 `localhost:5173` 等），不允许任意来源
- **上传资源**：`/uploads/**` 需登录（用户头像受保护）；前端 `<img>` 无法带 token，故用 `useAuthAvatar` 以带 token 的请求拉取 blob 展示。动作图走前端静态 `/exercises/**`，不涉及后端
- **密钥**：JWT secret / DB 口令一律走环境变量，禁止写入源码或配置文件；**唯一例外**：AI 使用的智谱 GLM key 为免费 API key，按用户要求内置为默认值（可用环境变量 `GLM_API_KEY` 覆盖），见第 11 章

## 11. AI 功能

- 已接入 **智谱 GLM**（OpenAI 兼容协议，模型 `glm-4.7-flash`）：`ai/AiService` 门面 + `ai/provider/AiProvider` SPI + `ai/dto`
- 接口：`POST /api/ai/chat`、`/api/ai/training-plan`、`/api/ai/diet-analysis`
- API Key：免费 key 内置为默认值（`application.yml` 中 `${GLM_API_KEY:<内置key>}`），可用环境变量 `GLM_API_KEY` 覆盖；key 无效/超限时接口返回 503，前端需优雅降级（不缓存假建议）
- 前端 AI 页的快捷操作全部基于真实统计/档案数据拼 prompt，不新增后端接口

## 12. 测试要求

- **前端**：`cd client && npm run typecheck`（vue-tsc）+ `npm run build`（typecheck + vite build）
- **后端**：`mvn package -DskipTests`（编译 + 打包）
- **QA 脚本**：`client/scripts/screenshot.mjs`（全部页面 × 多宽度截图诊断重叠/控制台错误）；`client/scripts/repro-comment.mjs`（评论输入回归）
- **手动冒烟**：登录（admin/admin123）→ 首页图表 → 训练打卡 → 饮食 → 身体 → 动作教学 → 分析 → AI → 个人中心 → 评论

## 13. 执行任务时的工作流程

1. **先理解**：读相关代码（页面 / 组件 / API / composables），复用已有能力，不重复造轮子
2. **改前确认**：涉及方向性/架构/安全决策时先与用户确认；涉及修改关键逻辑（训练双模式 payload、社区乐观更新、头像保护）需说明影响
3. **修改**：遵循本规范，小步改动
4. **验证**：`npm run typecheck` → `npm run build` → 启动后端 + 前端 → 浏览器 / puppeteer 验证改动生效、无控制台错误、无布局重叠
5. **提交**：按 Git 规则提交

## 14. 遇到问题时

- **环境限制**：本机 Docker（WSL）与全局 Maven 不可用，`docker compose` 跑不起来。实际可用环境：原生 **MySQL80**（库 `fitness`，root/root）+ **Memurai** 服务（Redis 兼容，6379）+ Edge（puppeteer 可截图）
- **运行后端**：`java -jar server/target/fitness-server-1.0.0-SNAPSHOT.jar --spring.profiles.active=dev`（默认账号 admin/admin123）
- **重新构建后端**：从阿里云镜像下载 Maven 后执行 `mvn package -DskipTests`（`~/.m2` 无缓存，需配 aliyun mirror 拉依赖）
- **查日志**：`/tmp/fitness-server.log`（后端 stdout）、`server/logs/`
- **原则**：先核实事实（读代码 / 验证接口）再动手；无法验证的内容如实说明，不臆断

## 15. Git 规则

- 提交信息用 `feat:` / `fix:` / `chore:` / `docs:` 前缀 + 中文简述
- **先提交 checkpoint 再做大清理**，避免大量改动无法回退
- 不提交 gitignored 产物：`server/uploads/`、`server/target/`、`client/dist/`、`node_modules/`、日志、`.snapshots/`、`.reasonix/`
- 敏感信息（口令、JWT secret、付费 API Key）绝不入库；**例外**：GLM 免费 API key 按用户决定内置（第 11 章）

## 16. AI 行为约束

- **不因视觉修改后端接口 / 数据结构**；不伪造或虚构数据 / 假建议
- **不删除或破坏现有功能**；训练打卡双模式提交的 payload 构造、社区点赞/收藏乐观更新逻辑为刻意保留，改动前必须说明
- 头像保持受保护（不做公开放行）；涉及放宽权限时需先向用户说明并由用户决定
- 改动前先探索理解，改动后如实报告（包括验证失败的情况）
- 复用优先于新建；避免重复代码与一次性脚本残留

## 17. 输出要求

- 使用中文
- 引用文件用仓库相对路径（如 `client/src/views/HomeView.vue`）
- 每个任务结束给出：改动文件清单、验证结果（通过/失败如实）、遗留事项
- 简明扼要，不堆砌过程性信息

---
> Source: [Jas-Wick/fitness-web](https://github.com/Jas-Wick/fitness-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
