## inferlens

> - InferLens 是面向自建 LLM 推理服务的轻量观测与调试工具。

# InferLens 协作约定

## 项目定位
- InferLens 是面向自建 LLM 推理服务的轻量观测与调试工具。
- 当前阶段优先建立“本地 vLLM CLI 闭环”，即从本机发请求到本机推理服务并拿到结果。

## 当前阶段目标
- 优先完成 `Go CLI -> 本地 vLLM -> 返回结果` 的最小可用链路。
- 暂不实现 Kubernetes、Grafana、MLOps 平台能力。
- 每个阶段都要求结果可运行、可验证、可演示。

## 技术约定
- 主语言使用 Go。
- v0 优先使用标准库，避免不必要的第三方依赖。
- 推理接口优先兼容 OpenAI 风格 API，以便适配 vLLM 及后续生态集成。
- 项目初期使用扁平目录结构，不过早引入 `internal/`。

## 目录约定
- `cmd/inferlens/`：CLI 入口。
- `config/`：配置加载与优先级处理。
- `client/`：推理服务客户端。
- `output/`：终端输出渲染。
- `metrics/`：基础指标采集与封装。
- `example/`：示例命令和样例输入。

## 开发原则
- 先做最小闭环，再扩展可观测性和云原生能力。
- 避免过早抽象，只有在需求已经稳定时再提炼通用层。
- 优先保留清晰的请求链路和易懂的代码结构。
- 默认先解决本地单机场景，再向 benchmark、监控、调度场景演进。
- 如果需要编写计划文档，统一写入 `.codex/plan/`，不散落到仓库其他位置。

## 里程碑
- `v0`：本地 CLI 调用 vLLM 并返回结果。
- `v0.1`：支持流式输出、重试和超时控制完善。
- `v0.2`：增加 benchmark 命令，统计延迟与吞吐。
- `v0.3`：接入 Prometheus 指标并配置 Grafana 看板。
- `v1`：支持 Kubernetes 调度与云原生部署场景。

---
> Source: [caffe-in/inferlens](https://github.com/caffe-in/inferlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
