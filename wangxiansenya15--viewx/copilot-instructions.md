## viewx

> "name": "ViewX 后端核心架构",

{
  "rules": [
    {
      "name": "ViewX 后端核心架构",
      "description": "定义了 ViewX 后端使用的语言、框架和分层规范。",
      "match": "**/src/main/java/com/flowbrain/viewx/**",
      "context": [
        "使用 Java 17+ 和 Spring Boot 3.",
        "严格遵循 Controller/Service/DAO 分层，Service 层是业务核心。",
        "使用 DTO 作为入参，VO 作为出参，Entity 对应数据库。"
      ]
    },
    {
      "name": "ViewX 统一返回规范",
      "description": "定义了 Controller 层的统一返回格式。",
      "match": "**/src/main/java/com/flowbrain/viewx/controller/**",
      "context": [
        "所有 Controller 接口必须使用 com.flowbrain.viewx.common.Result 作为返回类型。",
        "成功返回使用 Result.success()、Result.success(data)、Result.success(message, data)。",
        "失败返回使用 Result.badRequest()、Result.unauthorized()、Result.forbidden()、Result.notFound()、Result.serverError()。",
        "禁止使用 ResponseEntity<Map<String, Object>> 等非标准返回类型。",
        "Result 类已定义在 com.flowbrain.viewx.common.Result，包含 code、message、data 三个字段。"
      ]
    },
    {
      "name": "ViewX Servlet API 版本",
      "description": "定义了 Servlet API 的版本规范。",
      "context": [
        "项目使用 Spring Boot 3.x，必须使用 jakarta.servlet 而不是 javax.servlet。",
        "导入 HttpServletRequest 时使用 import jakarta.servlet.http.HttpServletRequest;",
        "所有 javax.servlet.* 导入都应替换为 jakarta.servlet.*"
      ]
    },
    {
      "name": "ViewX 高性能组件与 AI",
      "description": "定义了缓存、消息队列和 AI 集成的具体技术栈。",
      "context": [
        "使用 Redis 进行高性能计数和缓存。",
        "使用 RabbitMQ 进行异步任务解耦（如通知和行为日志）。",
        "AI 语义搜索依赖 PostgreSQL 的 pgvector 扩展。"
      ]
    },
    {
      "name": "排除非代码文件",
      "exclude": [
        "target/"
      ]
    },
    {
      "name": "回复语言偏好 (Response Language Preference)",
      "description": "提醒 AI 在所有代码生成、解释和交流中都使用中文。",
      "context": [
        "**重要提示：所有对我的问题的回答、代码解释、文档生成和建议，都必须使用流利的中文（简体）进行回复。**"
      ]
    }
  ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wangxiansenya15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
