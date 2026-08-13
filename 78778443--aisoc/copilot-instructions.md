## aisoc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> 📌 **重要记忆**：完整的项目需求背景、设计思路和开发历史请查看 `docs/project_session_history.md` 文件，包含从项目启动到当前阶段的所有会话内容。

## 项目概述
本仓库是AISOC（智能安全运营中心）开发视频教程项目，面向安全工程师教学如何开发一套完整的智能安全分析系统。

## 系统整体架构
```
用户自然语言查询 → 意图识别模块 → 数据获取模块 → 数据聚合模块 → LLM分析模块 → 运营事件存储 → Web控制台展示
```

### 核心层级
1. **数据源层**
   - HIDS：资产数据、告警数据、基线检查数据、漏洞数据
   - WAF：Web攻击事件数据
   - NIDS：网络攻击事件数据、五元组数据、出网域名数据
   - 蜜罐：蜜罐攻击日志数据

2. **数据存储层**
   - ClickHouse：存储全量日志、事件类时序数据
   - MySQL：存储系统配置、元数据、运营事件结果数据

3. **核心处理模块**
   - 意图识别模块：解析用户自然语言输入，识别查询意图，返回需要查询的表名和查询目标
   - 数据获取模块：根据意图自动生成SQL语句，从ClickHouse/MySQL中查询所需数据
   - 数据聚合模块：将多表、多来源的数据进行统一组装和格式化
   - LLM分析模块：对聚合后的安全数据进行分析，生成安全事件结论和处置建议

4. **展示层**
   - Web控制台首页：安全概览统计
   - 运营事件列表页：所有分析完成的安全事件列表
   - 事件详情页：展示完整分析链路，包含每个处理节点的输入输出数据，便于回溯和调试

## 技术栈
- 后端：ThinkPHP 8
- 数据库：MySQL 8.0 + ClickHouse
- LLM：字节跳动火山引擎豆包API

## 目录结构
```
├── code/             # ThinkPHP后端代码
│   ├── app/
│   │   ├── controller/  # 控制器
│   │   ├── model/       # 模型
│   │   └── service/     # 业务服务层
│   ├── config/          # 配置文件
│   └── public/          # 入口目录
├── sql/              # 数据库表结构定义
│   ├── mysql_schema.sql
│   └── clickhouse_schema.sql
├── frontend/         # Web控制台前端
├── docs/             # 教程文档和课件
└── demo-data/        # 模拟数据源和测试数据
```

## 常用命令
```bash
# 安装依赖
cd code && composer install

# 启动开发服务器
cd code && php think run

# 导入MySQL表结构
mysql -h 127.0.0.1 -u root -p luzhi-aisoc < sql/mysql_schema.sql

# 导入ClickHouse表结构
clickhouse-client -h 10.126.126.4 -d luzhi-aisoc < sql/clickhouse_schema.sql
```

## API接口
- `GET /` - 首页统计数据
- `POST /analyze` - 提交自然语言查询进行分析
  - 参数：query [string] 用户查询内容
- `GET /events` - 获取事件列表
  - 参数：page, limit, level (可选，过滤事件等级)
- `GET /event/:id` - 获取事件详情，包含所有处理节点日志

---
> Source: [78778443/aisoc](https://github.com/78778443/aisoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
