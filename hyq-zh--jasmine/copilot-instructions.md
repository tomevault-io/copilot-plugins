## jasmine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概况

Jasmine 是一个用 Go 编写的压测（负载测试）框架库，模块名 `jasmine`，单一扁平 package `jasmine`。仓库目前没有 `main` 包、没有测试文件、没有 CI 配置，也还没有任何 commit（`main` 分支为空）。设计上明显参考了 boomer（见 `NewStandaloneBoomer`）与 slidingwindow 限流库。

## 常用命令

```bash
go build ./...          # 编译
go vet ./...            # 静态检查
gofmt -l .              # 格式检查（列出未格式化文件）
go test ./...           # 运行测试（当前无测试文件）
go test -run TestXxx    # 运行单个测试
go test -race ./...     # 竞态检测，改动限流器/runner 的并发逻辑后建议执行
```

`gofmt -l .` 会列出全部 `.go` 文件，原因是仓库统一用 CRLF 行尾而 gofmt 要求 LF——**不是缩进排版问题**。别顺手跑 `gofmt -w`，那会重写每个文件的每一行，在这个尚无 commit 的仓库里会把真实改动完全淹没。

## 架构分层

自上而下四层，调用链为 `Perf.Run` → `localRunner.run` → `spawnWorkers` → `Limiter.Allow` → `Task.Fn`。

**配置层**（`task.go` + `parse.go`）
`YokaConfig` / `TestCase` / `CustomParam` 是命令行参数的承载结构。`CommandParse()` 手写遍历 `os.Args`，不使用 `flag` 包；无法识别的参数会被当作 `CustomParam` 键值对收集。`GetPodCount` 与 `CreateRateLimiter` 负责分布式场景下按 `podId`/`podNum` 把总阈值切分到各个 Pod（余数分配给前几个 Pod）。

**入口层**（`perf.go`）
`Perf` 是对外门面。`NewStandaloneBoomer(spawnCount, spawnRate, rateLimiter)` 构造实例，`Run(executeMode, executeCtl, tasks...)` 按 `mode` 分发。目前只实现了 `StandaloneMode`，`Mode` 枚举预留了扩展位（分布式模式尚未实现）。

**执行层**（`runner.go`）
`runner` 是基础结构，`localRunner` 通过嵌入复用它。两种执行模式差异很大：

- `DefaultMode` → `addCycleWorkers`：用 `sync.WaitGroup` 等待全部 goroutine 结束，然后 `cancel()` 并 sleep 10s。
- `TimerMode` → `addTimerWorkers`：按 `executeCtl`（`time.ParseDuration` 格式，如 `"30s"`）跑固定时长，到点后 `cancel()`、sleep 5s、**直接 `os.Exit(0)`**。这意味着 TimerMode 无法嵌入到需要继续存活的宿主进程里。

`getTaskToScale()` 按 `Task.Weight` 做加权随机选择；`safeRun()` 用 `recover` 兜住业务 panic 并把堆栈打到 stderr，保证单个任务崩溃不影响整体。

**限流层**（`slidingwindow.go` + `window.go`）
滑动窗口计数器是**唯一**的限流实现：`Limiter` 用前一窗口的计数按时间权重加权（`weight = (size - elapsed) / size`），得到比固定窗口更平滑的速率估计。原先并存的令牌桶实现（`ratelimiter.go` 的 `RateLimiter` 接口 / `StableRateLimiter` / `RampUpRateLimiter`）已删除，全链路统一走 `*Limiter`。

注意文件名与内容错位：`Limiter` 在 `slidingwindow.go` 里，`LocalWindow` 在 `window.go` 里。

关键语义——**超限不阻塞**。`Allow()` 超出阈值时立即返回 `false`，runner 的 `allowed()` 据此**跳过**本次任务、goroutine 直接退出，不会排队等待下个窗口。因此实际发出的请求数会低于 `spawnCount`。`Limiter` 自己持有窗口的 `StopFunc`，通过 `Limiter.Stop()` 释放；`runner.stopRateLimiter()` 在 DefaultMode 与 TimerMode 两条路径上都会调用（TimerMode 因为走 `os.Exit(0)`、defer 不执行，所以是显式调用）。`rateLimiter` 允许为 nil，表示不限速。

**集合点**（`rendezvous.go`）
`RendezvousPoint` 提供 `Done(delta)` / `Wait()` 用于多协程同步汇聚，`Wait` 是 10ms 轮询。

## 已知不一致（改动前请留意）

这些不是需要你顺手修的问题，但读代码时会被绊到：

- `Rendezvous` 接口声明的是 `Done()`，而 `RendezvousPoint.Done(delta int64)` 带参数，**并未实现该接口**；`NewRendezvousPoint` 也从没往 `RendezvousPoints` 全局 map 里写入过。
- 令牌桶删除后，`TestCase.RampUpPeriod` 已无实现载体（原先由 `RampUpRateLimiter` 承担爬坡）。若要恢复爬坡能力，需在 `Limiter` 上扩展递增 `limit` 的逻辑。
- `CommandParse` 在循环外只创建了一个 `yokaTestCase`，多个 `--case` 会共享同一个指针；且 `yokaTestCase` 永不为 nil，导致写入 `GlobalParams` 的分支不可达。
- `reduceWorkers(gapCount)` 只在 `gapCount == 0` 时才进入逻辑体，实际缩容不生效。
- 每个 worker goroutine 只执行任务**一次**就退出，不像 boomer 那样循环执行。`TestCase` 里的 `Duration` / `LoopNum` / `RampUpPeriod` 已被解析但 runner 尚未消费。
- `runner.state` 与相关状态常量已被注释掉，状态机未启用。
- `spawnRate` 被传递和保存，但 `startSpawning` 并未依据它做速率控制。

## 约定

- 代码注释与文档使用中文，导出标识符的 godoc 注释沿用英文（现有代码即为此混合风格，保持一致）。
- 并发状态一律走 `sync/atomic` 或 `sync.Mutex`，不要引入新的裸共享变量。

---
> Source: [hyq-zh/Jasmine](https://github.com/hyq-zh/Jasmine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
