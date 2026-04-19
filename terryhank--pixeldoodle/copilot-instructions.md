## pixeldoodle

> 1. 每当你推送时，连接服务器ssh terry@10.0.0.99 ，然后每次我让你推送按照这样的格式推送“版本号.年(例如26年).月日(例如316，3月16日)”，在服务器上执行执行：“1. 强制拉取最新版本

1. 每当你推送时，连接服务器ssh terry@10.0.0.99 ，然后每次我让你推送按照这样的格式推送“版本号.年(例如26年).月日(例如316，3月16日)”，在服务器上执行执行：“1. 强制拉取最新版本
   - 命令: git fetch https://gh-proxy.com/github.com/TerryHank/PixelDoodle
   - 命令: git reset --hard FETCH_HEAD
   - 效果: 完全覆盖本地所有文件，丢弃所有本地修改，同步到远程最新提交
    重启服务
   - 在 tmux main 会话中发送 Ctrl+C 停止旧进程
   - 执行 cd /home/terry/PixelDoodle && python3 main.py 重启服务”

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TerryHank) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
