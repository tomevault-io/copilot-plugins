## dianshu-mcp

> - 每次修改完后，需要 `go fmt ./...` 格式化 Go 源码文件

# Project Guidelines

## 本地开发规范

- 每次修改完后，需要 `go fmt ./...` 格式化 Go 源码文件
- 测试过程中产生的 build 中间文件，必要时删除
- Feature 变更使用分支开发
- 未同意前不推送到远程
- 变更流程：本地 review → 远程 PR review
- 不要过度设计，保持代码简洁
- 使用中文注释，专业名词可用英文

## 项目结构

```
dianshu-mcp/
├── main.go                  # 入口
├── server.go                # 应用容器
├── routes.go                # HTTP 路由（MCP + CORS）
├── mcp.go                   # MCP 工具注册（16 个工具）
├── config/config.go         # 统一配置
├── logger/logger.go         # 统一日志
├── handler/handler.go       # MCP 处理层
├── service/service.go       # 业务层
├── dianshu/                 # 典枢平台 HTTP 客户端
│   ├── api.go               # API 端点
│   ├── auth.go              # 微信扫码登录
│   ├── browser.go           # go-rod 浏览器
│   ├── cookies.go           # Cookie 持久化
│   ├── types.go             # 数据类型
│   └── dataset_types.go     # 数据集类型
├── pkg/                     # 独立 SDK 模块（与业务解耦）
│   ├── chain/               # 链上操作
│   ├── crypto/              # 加密模块
│   ├── kms/                 # KMS 集成
│   ├── pipeline/            # 下载管线
│   └── sdk/                 # 数据 API SDK
├── output/                  # 输出目录
└── .skill/dianshu/  # Agent Skills
```

## MCP 工具清单（16 个）

| 工具 | 功能 |
|------|------|
| `check_login_status` | 检查登录状态 |
| `get_login_qrcode` | 微信扫码登录（返回 PNG 二维码） |
| `delete_cookies` | 清除 cookies，切换账号 |
| `list_orders` | 查询订单列表 |
| `list_downloads` | 列出已购买的可下载数据 |
| `download_order` | 下载并解密数据文件 |
| `list_purchased_apis` | 列出已购买的 API |
| `get_api_detail` | 获取 API 参数列表 |
| `call_api` | 调用数据 API（自动加解密） |
| `search_datasets` | 搜索典枢数据集 |
| `dataset_detail` | 获取数据集详情 |
| `homepage_recommend` | 首页推荐数据集 |
| `my_datasets` | 我发布的数据集 |
| `get_my_profile` | 获取账号资料 |
| `get_my_wallet` | 获取钱包余额 |
| `list_wallet_transactions` | 钱包交易明细 |

## 技术栈

- 语言：Go 1.22+
- Web 框架：Gin
- MCP SDK：`github.com/modelcontextprotocol/go-sdk`
- 浏览器自动化：`github.com/go-rod/rod`
- MCP 传输：Streamable HTTP（`:18061`）

---
> Source: [YeeZTech/dianshu-mcp](https://github.com/YeeZTech/dianshu-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
