## port-lite

> 这个仓库是 PortLite，一个用 Wails v3 + Vue 写的 Windows 本地端口查看器。

# AGENTS.md

这个仓库是 PortLite，一个用 Wails v3 + Vue 写的 Windows 本地端口查看器。

后续在这个仓库里改代码时，优先遵守下面这些约定。

## 项目定位

- 这是一个给开发者排查本地端口占用的小工具，不是完整的系统进程管理器。
- 功能要克制，优先解决“哪个端口被哪个进程占了”这个核心问题。
- 新功能最好能服务公众号文章里的真实场景，比如端口占用、开发服务启动失败、本地调试冲突。

## 技术栈

- 桌面框架：Wails v3
- 后端：Go
- 前端：Vue 3 + TypeScript + Vite
- 主要平台：Windows

## 开发约定

- Windows 端口查询逻辑放在 `portservice_windows.go`。
- 非 Windows 平台只保留占位实现，不要假装已经完整支持。
- 前端尽量保持轻量，不引入大型 UI 组件库。
- 结束进程属于危险操作，必须保留明确确认，不要做一键批量结束。
- PID `0`、PID `4`、当前应用自身进程必须继续保护。
- 涉及进程路径、用户名、公司目录的截图或文章素材要先脱敏。

## 验证命令

修改后优先跑：

```powershell
cd frontend
npm install
npm run build
cd ..

go test ./...
wails3 task build
```

如果只是改 README 或文章素材，可以不跑完整构建，但要说明没有运行。

## 公众号文章配套

这个项目后续会作为“开源端口查看器”的文章源码使用。

文章表达上不要写成标准教程，更适合按真实过程来写：

- 为什么想做这个工具。
- 平时怎么查端口。
- 做完后解决了什么麻烦。
- 哪些功能先没做。
- 当前 exe 体积和测试环境。

文章里如果出现本机路径、用户名、微信账号、公司项目名，必须先脱敏。

---
> Source: [fengfengzhidao/port_lite](https://github.com/fengfengzhidao/port_lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
