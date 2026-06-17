## webtrailai

> 本文件只定义全项目通用规则。进入具体目录后，优先遵循更近一级的 `AGENTS.md`。

# WebTrailAI 协作规范

本文件只定义全项目通用规则。进入具体目录后，优先遵循更近一级的 `AGENTS.md`。

## 项目介绍
本项目实现前端浏览器插件，在用户浏览网页后，将网页 url 和标题保存，并且发送到后端。后端负责创建登录用户，并且接收用户浏览的网页信息，同时返回list， 后端每日会生成 AI解读，解读上一日用户浏览的所有网页信息汇总，并且给用户建议。


## 适用范围

- 根目录规则适用于整个仓库。
- `extension/` 的前端插件规范以 [extension/AGENTS.md](/Users/johnny/wwwroot/person/WebTrailAI/extension/AGENTS.md) 为准。
- `server/` 的 Go 后端规范以 [server/AGENTS.md](/Users/johnny/wwwroot/person/WebTrailAI/server/AGENTS.md) 为准。

## 目录边界

- `extension/` 只放浏览器插件前端代码与静态资源。
- `server/` 只放 Go 后端服务代码、配置、测试和本地文件数据库。
- 跨目录改动必须有明确原因，避免把前端逻辑写进后端，或把后端约束硬编码到前端。

## 变更原则

- 先读现有实现，再改代码；优先复用现有结构、命名和依赖。
- 不凭空引入新的框架、构建工具、目录层级或大规模重构。
- 只改当前任务需要的内容，顺手修复必须和当前问题直接相关。
- 涉及接口、配置、存储字段、鉴权规则的调整，必须同步检查调用方和使用方。
- 删除失效代码、失效配置和重复逻辑，不保留无意义的兼容分支。

## 提交前检查

- 优先执行项目里已经存在的命令，不编造仓库里不存在的 `lint`、`build`、`test` 流程。
- 至少验证本次改动影响到的那个子项目；如果同时改了 `extension/` 和 `server/`，两边都要验证。
- 无法完成自动验证时，要在结果说明里明确写出未验证项、原因和建议补验方式。

## 文档维护

- 新增目录规则时，优先写入对应目录的 `AGENTS.md`，不要把实现细节堆到根目录。
- 如果真实启动命令、测试命令、目录职责发生变化，要同步更新对应层级的 `AGENTS.md`。

---
> Source: [gwyy/WebTrailAI](https://github.com/gwyy/WebTrailAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
