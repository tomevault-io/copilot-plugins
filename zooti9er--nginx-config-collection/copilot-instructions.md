## nginx-config-collection

> Thinking in English, speaking in Chinese.

# CLAUDE.md
Thinking in English, speaking in Chinese.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是 Homebrew 安装的 Nginx 1.29.1 的配置目录，位于 `/opt/homebrew/etc/nginx/`。该目录包含了 Nginx 服务器的所有配置文件。

## 常用命令

### Nginx 服务管理
```bash
# 启动 Nginx 服务
brew services start nginx

# 停止 Nginx 服务
brew services stop nginx

# 重启 Nginx 服务
brew services restart nginx

# 重新加载配置（不中断服务）
nginx -s reload

# 查看服务状态
brew services list | grep nginx

# 查看 Nginx 进程
ps aux | grep nginx
```

### 配置文件测试和调试
```bash
# 测试配置文件语法
nginx -t

# 检查配置文件语法并显示完整配置
nginx -T

# 查看编译模块和配置参数
nginx -V
```

### 日志管理
```bash
# 实时查看错误日志
tail -f /opt/homebrew/var/log/nginx/error.log

# 实时查看访问日志
tail -f /opt/homebrew/var/log/nginx/access.log

# 查看最近的错误
tail -n 50 /opt/homebrew/var/log/nginx/error.log
```

### 直接运行 Nginx（调试用）
```bash
# 直接运行 Nginx（前台模式，适合调试）
/opt/homebrew/opt/nginx/bin/nginx -g daemon\ off\;

# 指定配置文件运行
nginx -c /opt/homebrew/etc/nginx/nginx.conf

# 停止 Nginx
nginx -s stop
```

## 配置架构

### 主要配置文件
- **nginx.conf** - 主配置文件，定义全局设置和默认服务器
- **servers/** - 包含所有虚拟主机配置文件
- **mime.types** - MIME 类型映射配置
- **fastcgi_params** - FastCGI 参数配置
- **scgi_params** - SCGI 参数配置
- **uwsgi_params** - uWSGI 参数配置

### 服务器配置组织
Nginx 通过 `include servers/*;` 自动加载 `/opt/homebrew/etc/nginx/servers/` 目录下的所有配置文件，每个 `.conf` 文件代表一个虚拟主机。

### 当前配置结构
1. **默认服务器** (nginx.conf:35-79) - 监听 8080 端口，服务本地静态文件
2. **Ollama 代理** (servers/ollama_proxy.conf) - 监听 11436 端口，代理 Ollama API 请求到本地 11435 端口

### 服务器配置加载机制
Nginx 通过主配置文件末尾的 `include servers/*;` (nginx.conf:116) 自动加载 `/opt/homebrew/etc/nginx/servers/` 目录下的所有配置文件。每个 `.conf` 文件代表一个独立的虚拟主机配置。

## 关键路径

### 安装路径
- **Nginx 二进制**: `/opt/homebrew/bin/nginx`
- **配置目录**: `/opt/homebrew/etc/nginx/`
- **Web 根目录**: `/opt/homebrew/var/www` (实际使用的是 nginx.conf 中定义的 html 目录)
- **日志目录**: `/opt/homebrew/var/log/nginx/`
- **PID 文件**: `/opt/homebrew/var/run/nginx.pid`

### 临时文件路径
- 客户端请求体临时文件: `/opt/homebrew/var/run/nginx/client_body_temp`
- 代理临时文件: `/opt/homebrew/var/run/nginx/proxy_temp`
- FastCGI 临时文件: `/opt/homebrew/var/run/nginx/fastcgi_temp`
- uWSGI 临时文件: `/opt/homebrew/var/run/nginx/uwsgi_temp`
- SCGI 临时文件: `/opt/homebrew/var/run/nginx/scgi_temp`

## 编译模块和配置参数

### 编译模块
Nginx 包含以下编译的模块：
- **HTTP 核心模块**: addition, auth_request, dav, degradation, flv, gunzip, gzip_static, mp4, random_index, realip, secure_link, slice, ssl, stub_status, sub, v2, v3
- **邮件模块**: mail, mail_ssl
- **流模块**: stream, stream_realip, stream_ssl, stream_ssl_preread
- **其他**: PCRE JIT, IPv6 支持

### 关键配置参数
- **前缀路径**: `/opt/homebrew/Cellar/nginx/1.29.1`
- **配置文件路径**: `/opt/homebrew/etc/nginx/nginx.conf`
- **PID 文件路径**: `/opt/homebrew/var/run/nginx.pid`
- **临时文件路径**: `/opt/homebrew/var/run/nginx/*_temp`
- **日志路径**: `/opt/homebrew/var/log/nginx/`
- **SSL 支持**: OpenSSL 3.5.2
- **调试模式**: 已启用
- **兼容性模块**: 已启用

## 开发规范

### 配置文件格式
- 使用标准 Nginx 配置语法
- 虚拟主机配置文件命名: `{service}_proxy.conf`
- 每个虚拟主机配置应包含完整的 server 块
- 代理配置需要设置适当的头部信息

### 端口使用
- 默认端口: 8080 (避免需要 sudo)
- Ollama 代理: 11436 (配置文件) + 11435 (实际 Ollama 服务)
- 生产环境建议使用标准端口 (80/443)

### 代理配置模式
```nginx
server {
    listen 端口;
    server_name 域名;

    location / {
        proxy_pass http://后端地址;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Ollama 代理特定配置
当前 Ollama 代理配置包含多个 location 块，处理不同的 API 路径：
- `/v1/` - 代理 OpenAI 兼容的 API
- `/api/tags` - 代理模型列表接口
- `/api/` - 代理其他 Ollama API 路径

### 代理配置最佳实践
1. **路径匹配优先级**: 精确匹配 (`=`) > 前缀匹配 (`^~`) > 正则匹配 (`~`) > 普通前缀匹配
2. **proxy_pass 路径处理**:
   - 带 `/` 结尾：去除 location 匹配部分后附加
   - 不带 `/` 结尾：完整替换 location 匹配部分
3. **头部传递**: 始终传递 `X-Real-IP`、`X-Forwarded-For`、`X-Forwarded-Proto` 以保持客户端信息
4. **超时设置**: 根据后端服务特性设置合适的 `proxy_connect_timeout` 和 `proxy_read_timeout`

## 注意事项

1. **权限问题**: Nginx 运行时需要适当的文件权限，特别是 PID 文件和日志文件
2. **端口冲突**: 确保配置的端口未被其他服务占用
3. **配置测试**: 每次修改配置后，务必运行 `nginx -t` 测试语法
4. **服务重启**: 配置更改后需要重启服务使更改生效
5. **日志查看**: 错误日志位于 `/opt/homebrew/var/log/nginx/error.log`

## 调试和故障排除

### 常见问题排查
1. **配置语法错误**: 使用 `nginx -t` 检查语法，查看错误日志定位具体问题
2. **端口占用**: 使用 `lsof -i :端口号` 检查端口是否被占用
3. **权限问题**: 确保 Nginx 进程有权限访问配置文件、日志目录和静态文件
4. **代理连接失败**: 检查后端服务是否正常运行，网络连接是否正常
5. **SSL 证书问题**: 确保证书文件路径正确，权限适当，未过期

### 性能监控
```bash
# 查看 Nginx 状态（需要启用 stub_status 模块）
curl http://localhost:8080/nginx_status

# 监控连接数
netstat -an | grep :8080 | wc -l

# 查看进程资源使用
top -p $(pgrep nginx | head -1)
```

---
> Source: [ZooTi9er/nginx-config-collection](https://github.com/ZooTi9er/nginx-config-collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
