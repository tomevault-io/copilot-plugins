## rv-139login

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目用途

模拟 `mail.10086.cn`（中国移动 139 邮箱）的图形验证码识别、短信验证码发送、登录及后续数据请求流程。

## 常用命令

```bash
# 安装依赖（建议使用虚拟环境）
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 运行完整登录流程
python3 captcha_verify_sim.py

# 自动使用默认值（跳过大部分交互提示）+ 指定手机号
python3 captcha_verify_sim.py --auto-defaults --phone 13800138000

# 开启 DEBUG 日志
python3 captcha_verify_sim.py --log-level DEBUG

# 单独调试 OCR（读取项目根目录下的 verifyCodeImg.jpg）
python3 captcha_recognizer.py --log-level DEBUG
```

## 架构

### `captcha_verify_sim.py` — 主流程

`CaptchaFlowClient` 是核心类，`run()` 方法执行以下交互流程：

1. **Cookie 预热** — GET `mail.10086.cn`，从重定向中提取初始 Cookie 和 `cguid`
2. **获取验证码** — GET 验证码图片，从 `Set-Cookie` 中提取 `agentid`
3. **OCR 识别** — 识别验证码中需要点击的目标文字
4. **checkNewPictureCode** — POST XML 到 `login:checkNewPictureCode`，携带 verifyCode
5. **sendSmsCode** — POST XML 到 `login:sendSmsCode`，从响应 Cookie 中提取 `UUIDToken`
6. **Login.ashx** — POST form-urlencoded 登录请求，跟随重定向，提取 `sid`
7. **getMainData** — 用 `sid` 请求 `user:getMainData`

关键实现细节：
- **RSA 加密**：计算 `loginName` 时优先使用 `cryptography` 库，不可用时回退到 `openssl pkeyutl` 子进程
- **密码摘要**：`SHA1("fetion.com.cn:" + 明文密码)`，不是原始密码
- **verifyCode**：JSON 载荷 `{"k": 目标文字, "p": [点击坐标], "c": md5(agentid首段 + 坐标和)}` 经过双重 URL 编码
- **Cookie 处理**：手动解析/合并 `Set-Cookie` 响应头和 Cookie 字符串；`CookieJar` 仅在 `Login.ashx` 中用于捕获重定向时的 Cookie
- 每次 HTTP 请求/响应都会保存到 `tmp/` 下的时间戳子目录中，便于排查

### `captcha_recognizer.py` — OCR 模块

`CaptchaRecognizer` 封装 PaddleOCR，具有以下定制行为：
- **多字拆分**：当 PaddleOCR 检测到多字框时，沿长轴方向将包围多边形拆分为逐字的子多边形
- **固定文本过滤**：忽略识别结果中的 `"139邮箱"`（验证码中的固定背景文字）
- **环境隔离**：将 `HOME`、`PADDLE_HOME`、`XDG_CACHE_HOME` 重定向到 `tmp/.paddle_cache/`，避免模型文件污染用户主目录
- `find_target_centers()` 是主入口 — 运行 OCR，构建文字到中心点的映射，返回目标文字中每个字的点击坐标
- 重试机制：`resolve_points_with_retry()` 在识别失败时自动重新请求验证码并重试（最多 5 次）

### `log_utils.py`

配置根 logger，格式为 `时间 级别 [文件名:行号] 消息`。支持 `LOG_LEVEL` 环境变量和 `--log-level` 命令行参数。

### `list_files_with_size.py`

无关工具脚本，用于列出文件大小并按大小分组生成 7z 压缩命令。不属于登录流程。

---
> Source: [jellyqwq/rv-139Login](https://github.com/jellyqwq/rv-139Login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
