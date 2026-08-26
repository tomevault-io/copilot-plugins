## yqad

> - 除 openspec 的 .md 外，新生成的 .md 文件统一放到 docs 目录下

## 规则

- 每次回答前先打招呼："您好，哥。"
- 除 openspec 的 .md 外，新生成的 .md 文件统一放到 docs 目录下
- openspec 的所有 .md 必须使用中文
- 接口命名风格必须与当前项目保持一致
- 有不确定的点，必须主动发起询问
- 实现功能前先调研方案，可列举对比后再实施
- 简洁至上：每次变更尽可能简单，仅改动必要代码
- 每次测试完后 kill 测试进程，不占用端口

## 生产环境连接信息

- MySQL: `192.168.50.10:3306`（x5 本地 MariaDB 11），用户 `root`，密码 `Wfw7539148@`，库名 `yqad_prod_db`
- Redis: `192.168.50.10:6379`（x5 本地 Redis 7），db `1`，keyPrefix `prod:`
- ChromaDB: `http://192.168.50.10:8000`（x5 本地）
- 生产部署服务器（x5-server）: `192.168.50.10`，用户 `root`，密码 `Wfw7539148@`
  - Docker 数据目录: `/opt/docker/yqad/`
  - 编排文件: `/opt/docker/docker-compose.yml`
  - 网络模式: 尽量用bridge，特殊情况下可使用host
  - 服务端口: 3080（3000 被 Nginx Proxy Manager 占用）
  - 访问地址: `http://192.168.50.10:3080`
  - 容器内数据库连接使用服务名: `mysql`、`redis`、`chromadb`（docker compose 内部 DNS）

## 构建与部署

- 本地启动服务使用 3000 端口；如端口被占用，先 kill 占用进程再启动
- 本地启动服务、打包镜像一律连接生产数据库
- 打包生产镜像时必须指定 `--platform linux/amd64`
- x5-server 上构建镜像需加 `--network host`（容器默认桥接网络无法访问外网）
- 打包前须确保 dist 为最新编译产物；如无法确认，先 `rm -rf dist` 再 `npm run build`
- 打包镜像时确保所有运行时依赖完整，不可遗漏（特别注意 Python 脚本依赖和 native 模块）
- 打包生产镜像导出固定为 `~/Documents/workspace/krio/yqad-latest.tar`，重名直接覆盖
- **日常部署（增量）**：`./scripts/deploy.sh` — 只编译+同步代码到容器内并重启（~15s）
- **全量部署**：`./scripts/deploy.sh --full` — 重建镜像（修改了 package.json / Dockerfile / 系统依赖时使用）
- Dockerfile 分层策略：系统依赖 → Python 依赖 → npm 依赖 → Playwright → 入口脚本 → **应用代码（最后）**，确保日常构建命中缓存

## 常见部署问题

- `Cannot GET /`：通常是 `dist/web/public/` 目录缺失导致静态文件服务失败。原因是 tsc 编译报错后 build 脚本中的 `cp -r src/web/public/* dist/web/public/` 未执行。部署前必须确认 `dist/web/public/` 目录存在且包含 html 文件。
- `No module named 'playwright'`：Python 脚本使用了 playwright，Dockerfile 中 pip install 必须包含 `playwright` 包。Node.js 的 `npx playwright install` 只安装浏览器二进制，不提供 Python 模块。
- 容器内所有包管理器必须使用国内镜像：
  - apt: 阿里云 `mirrors.aliyun.com`（sed 替换 sources.list）
  - npm: 淘宝镜像 `registry.npmmirror.com`
  - pip: 清华镜像 `pypi.tuna.tsinghua.edu.cn/simple`
  - Playwright 浏览器下载: `PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright`
- 本地配置变更不生效（数据库不同步）：本地开发未设 `NODE_ENV=production` 时，MySQL 会连 `yqad_db`（test 库）而非 `yqad_prod_db`，ChromaDB collection 前缀也会用 `dev_` 而非 `prod_`。已修复所有连接管理器统一逻辑：优先 `NODE_ENV` → yaml `env` 字段 → 默认 test。务必确保 `config/default.yaml` 中 `env: "production"`。
  - MySQL: `mysql-connection-manager.ts` — 决定连 `yqad_db` 还是 `yqad_prod_db`
  - Redis: `redis-config-loader.ts` — 决定用 db 0（test）还是 db 1（prod），keyPrefix `test:` vs `prod:`
  - ChromaDB: `chroma-connection-manager.ts` — 决定 collection 前缀 `dev_` vs `prod_`

## 关键规则

- openspec 归档时，如存在 delta specs 则直接归档到 specs，无需二次确认
- openspec 归档后，须确认归档的 changes 目录下无遗留文件

---
> Source: [aochuanguying/yqad](https://github.com/aochuanguying/yqad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
