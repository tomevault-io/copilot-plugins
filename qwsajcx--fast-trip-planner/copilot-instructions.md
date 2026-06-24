## fast-trip-planner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

通用旅行规划助手 — 基于高德地图 API 和 flyai CLI 的 Python 命令行工具。通过更换配置和数据模板即可支持任意目的地，核心库（`lib/`）完全与目的地无关。

## 目录结构

```
trip/
├── trip-planner.py         # 交互式菜单主入口
├── config.yaml             # API Key + 偏好配置
├── lib/                    # 通用核心库（与目的地无关）
│   ├── geo.py              # 高德地理编码（地址→坐标）+ 坐标验证
│   ├── routes.py           # 高德路线计算（驾车/步行距离+时长）
│   ├── matrix.py           # 距离矩阵生成、打印、邻近查找、导出
│   ├── transport.py        # flyai CLI 火车票/机票查询（subprocess）
│   ├── hotel.py            # flyai CLI 酒店搜索（subprocess）
│   ├── hotel_location.py   # 酒店最优位置（景点重心 + Haversine 距离）
│   └── storage.py          # JSON 数据持久化（save/load）
├── scripts/
│   ├── update_trip.py      # 旅行途中 CLI：add/visit/status/matrix/nav
│   └── add_location.py     # 地点添加（单个/批量 JSON 导入）+ 坐标验证
├── templates/
│   └── locations_template.json  # 当前目的地（扬州）的景点预设模板
└── data/                   # 运行时数据（JSON，与当前行程绑定）
    ├── trip_state.json
    ├── routes.json
    └── locations.json
```

## 架构说明

**数据流（5 阶段）**: 地点模板导入 → 批量地理编码 → 路线距离计算 → 距离矩阵分析 → 交通/酒店查询 → 酒店位置优化 → 行程攻略输出

**lib/ 模块均为目的地无关**，通过输入数据（地点列表、坐标、日期）驱动行为：
- `geo.py` / `routes.py` 直接调用高德 HTTP API（`urllib.request`，无第三方依赖）。**地址必须精确到门牌号**——地理编码会返回 `level` 字段指示精度，`门址` 级别才可靠，`道路`/`兴趣点` 级别会触发警告。
- `transport.py` / `hotel.py` 通过 `subprocess` 调用 flyai CLI
- `hotel_location.py` 使用 Haversine 公式计算球面距离，不依赖外部 API
- `storage.py` 对 `data/` 目录读写 JSON，自动添加 `updated_at` 时间戳

**核心数据结构**:
- `trip_state.json`: `{trip_name, city, dates{start,end}, travelers[], transport{outbound,return}, hotel{name,brand,room_type,rooms,check_in,check_out,nights,address,status}, locations[], visit_history[]}`
- 每个 location: `{name, type, address, coordinate("lon,lat"), geocode_level, formatted_address, priority(must|optional), status, visit_order, notes}`
- 每条 route: `{origin, destination, distance_km, duration_min, origin_coord, dest_coord, mode(driving|walking)}`

**导航策略**: 从当前位置出发 → 筛选未访问非 transport 地点 → must 优先于 optional → 同优先级按距离升序。路线查找支持双向回退：优先查 `(current, dest)`，找不到时回退到 `(dest, current)`。

**关键阈值**: 步行 ≤1.5km/20min，邻近景点 ≤2.0km（可步行串联），API 请求间隔 200ms

**API 限流处理**: 基于高德官方 infocode 的重试机制（`lib/geo.py` 的 `api_call_with_retry`）。QPS 类限流（10004/10014/10015/10019/10020/10021）自动重试 3 次（1s→3s→6s 渐进等待）；日配额超限（10003）或 IP 封禁（10010）立即放弃。地理编码和路线计算均使用此机制。

**路线计算过滤** (两重):
1. 同类型 `food` 之间不计算（不会连着去两家餐厅）
2. 已访问地点不参与新路线计算（旧数据保留，避免无效 API 调用）
`batch_routes()` 和 `cmd_add()` 均遵循这些规则。

**地址精度验证**: `geocode()` 返回 `level` 字段，`check_precision()` 按精度级别判定——`门址` 通过，`道路`/`兴趣点` 触发警告。添加新地点时展示格式化地址和精度级别。

## 常用命令

```bash
# 依赖安装
pip install pyyaml

# 交互式主程序
python trip-planner.py

# 旅行途中操作
python scripts/update_trip.py status              # 查看行程进度
python scripts/update_trip.py visit <景点名>       # 标记已访问
python scripts/update_trip.py nav <当前位置>       # 实时导航推荐
python scripts/update_trip.py add <名称> <类型> <地址> [备注]  # 添加新地点
python scripts/update_trip.py matrix               # 打印距离矩阵

# 地点管理
python scripts/add_location.py <名称> <类型> <地址> [优先级] [备注]
python scripts/add_location.py --file templates/locations_template.json
```

## 更换目的地

要为新目的地复用此工具，只需修改：
1. `config.yaml` 的 `trip` 段落 — 修改 `name`、`city`、`travelers` 等字段
2. `templates/locations_template.json` — 新目的地的景点预设
3. 删除 `data/` 旧行程数据，重新初始化
4. `lib/` 模块和所有脚本无需修改

## 安全注意事项

- `config.yaml` 包含真实 API Key，**切勿提交到版本控制**
- `.gitignore` 已排除 `config.yaml`，建议将 API Key 迁移到环境变量

---
> Source: [qwsajcx/fast-trip-planner](https://github.com/qwsajcx/fast-trip-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
