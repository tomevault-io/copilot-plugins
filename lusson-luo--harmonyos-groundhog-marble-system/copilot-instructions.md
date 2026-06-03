## harmonyos-groundhog-marble-system

> marble-name-service/     # 新服务：大理石名称管理（Gin + SQLite）

# HarmonyOS 大理石系统

## 项目结构

```
marble-name-service/     # 新服务：大理石名称管理（Gin + SQLite）
backup/                  # 老服务：大理石后台管理（GoFrame + SQLite，线上运行）
front-manager/           # 前端管理台（Vue 3 + Element Plus）
TbsHarmonyOS/            # 鸿蒙 App
front-mini-programe/     # 微信小程序
```

## 新服务 marble-name-service

基于 Go + Gin 的轻量 CRUD 服务，与老服务共用 SQLite 数据库。

### 目录结构

```
marble-name-service/
  main.go                 -- 入口，端口 8001
  internal/
    db/sqlite.go          -- SQLite 连接
    handler/marble_name.go-- CRUD 接口实现
    model/marble_name.go  -- 请求/响应结构体
```

### API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/marble-name/list` | 分页列表（参数：pageNo, pageSize, name） |
| POST | `/api/marble-name/add` | 新增（自动设置 sort_order = MAX+1） |
| POST | `/api/marble-name/update` | 修改 |
| DELETE | `/api/marble-name/delete` | 删除（参数：id） |
| POST | `/api/marble-name/move-up` | 上移（参数：id，交换 sort_order） |
| POST | `/api/marble-name/move-down` | 下移（参数：id，交换 sort_order） |
| GET | `/wx-api/marble/name/list` | 小程序接口（分页，按 sort_order 排序） |

### 排序功能

- 表字段：`sort_order INTEGER`，新数据自动取 `MAX(sort_order)+1`
- 列表默认按 `sort_order, id` 升序排列
- 上移/下移通过交换两条记录的 sort_order 值实现
- 前端操作列显示上移/下移按钮，首行隐藏上移，末行隐藏下移

### 数据库

```sql
CREATE TABLE marbles_name (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    tenant_id INTEGER NOT NULL,
    name TEXT,
    picture_url TEXT,
    sort_order INTEGER DEFAULT 0
);
```

- 新服务 DSN 使用 `?_journal_mode=WAL&_busy_timeout=5000` 提高并发安全性
- 启动时自动执行 `ALTER TABLE ADD COLUMN` migration（幂等）
- 存量数据 `sort_order = id` 初始化

## 前端 front-manager

Vue 3 + Element Plus + TypeScript + Vite。

### 启动

```bash
cd front-manager && npm install --legacy-peer-deps --ignore-scripts && npm run dev
```

### 代理配置

| 代理路径 | 目标 | 说明 |
|----------|------|------|
| `/saas-api` | `http://newmanager.lusson.cn` | 老服务（线上） |
| `/api/marble-name` | `http://127.0.0.1:8001` | 新服务（本地） |

## 本地开发

需要同时启动两个服务：

1. **新服务**（大理石名称 CRUD）
   ```bash
   cd marble-name-service && go run main.go
   ```

2. **前端**
   ```bash
   cd front-manager && npm run dev
   ```

3. 浏览器打开 `http://localhost:5173`

老服务部署在线上 `http://newmanager.lusson.cn`，本地不需要启动。

## 依赖安装

前端首次运行需安装依赖（由于 husky 兼容性问题需跳过 prepare 脚本）：

```bash
cd front-manager && rm -rf node_modules && npm install --legacy-peer-deps --ignore-scripts
```

---
> Source: [lusson-luo/HarmonyOS-groundhog-marble-system](https://github.com/lusson-luo/HarmonyOS-groundhog-marble-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
