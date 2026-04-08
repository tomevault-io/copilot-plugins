## etl-neo4j-health-portrait

> 本项目是一个基于 **Python + Neo4j** 的医疗健康画像数据 ETL 工程。从 SQL Server 的 `ai_patients` 表获取患者 ID，调用大数据平台 API 拉取患者健康画像数据，经图谱建模后写入 Neo4j，最终通过 Flask REST API 对外提供查询服务。

# AGENTS.md — 健康画像 ETL 项目 AI 协作指南

## 项目概述

本项目是一个基于 **Python + Neo4j** 的医疗健康画像数据 ETL 工程。从 SQL Server 的 `ai_patients` 表获取患者 ID，调用大数据平台 API 拉取患者健康画像数据，经图谱建模后写入 Neo4j，最终通过 Flask REST API 对外提供查询服务。

---

## 代码库结构速查

```
etl_neo4j/
├── run_api.py                       # ★ API 服务独立启动入口
├── main.py                          # ★ ETL 主入口（增量执行、重试）
├── core/                            # 全局通用组件
│   ├── config.py                    # 全局配置（数据库、API、调度参数）
│   ├── logger.py                    # 日志处理
│   └── db.py                        # Neo4j 单例连接（线程安全）
├── api/                             # REST API 包模块
│   ├── __init__.py                  # Flask App 注册
│   └── routes.py                    # API 路由节点实现
├── etl/                             # 数据提取与转换逻辑
│   ├── extractors/                  # 从外部抓取数据
│   │   ├── health_portrait_api.py   # 大数据平台 HTTP 客户端
│   │   └── sqlserver.py             # SQL Server 客户端
│   ├── core/                        # ★ 图谱核心构建模块
│   │   └── etl_patient.py           # 图谱节点/关系的 Cypher 写入逻辑 
│   └── processors/                  
│       └── health_portrait.py       # 协调 API 调用与事务提交
├── scheduler/                       # 批处理调度
│   ├── job_manager.py               # 批量调度 + 错误队列
│   └── scheduler.py                 # 定时调度器
├── data/                            # 动态产生与运行时数据
│   ├── state/
│   │   └── etl_state.json           # 增量记录（上次成功时间戳）
│   └── samples/                     # 测试用 JSON 样例
├── tests/                           # 单元与集成测试脚本集
├── scripts/                         # 部署/环境验证运维工具
└── docs/                            # 文档说明
```

---

## 数据流与核心调用链

```
main.py
  └─ SQLServerConnection.load_patient_ids(last_update_time)   # 增量拉取患者 ID
  └─ JobManager.process_batch(empi_list)
       └─ ThreadPoolExecutor → _process_single(empi)
            ├─ HealthPortraitAPI.get_health_portrait(empi)     # 调用大数据平台
            └─ HealthPortraitProcessor.process(patient_data)
                 └─ session.execute_write(import_patient_data_from_json, data)
                      └─ etl_patient.py 中各 import_* 函数（Cypher MERGE 写入）
```

写入完成后，`etl_state.json` 记录本次运行时间戳，供下次增量查询使用。

---

## 图谱数据模型（节点与关系）

### 主要节点标签

| 标签 | 说明 | 关键属性 |
|---|---|---|
| `Patient` | 患者 | `patientId`, `name`, `birthDate`, `gender`, `idType`, `idValue` |
| `Encounter` | 就诊记录 | `encounterId`, `encounterType`, `typeName`, `visitStartTime` |
| `Hospital` | 医院 | `hospitalId`, `name` |
| `Department` | 科室 | `departmentId`, `name` |
| `Provider` | 医生 | `providerId`, `name` |
| `Condition` | 疾病/诊断 | `code`, `name` |
| `Allergen` | 过敏原 | `name`, `allergenType` |
| `LabTestReport` | 检验报告 | `reportId` |
| `LabTestItem` | 检验项目 | `name`, `code` |
| `Examination` | 检查记录 | `reportId`, `timestamp` |
| `PastMedicalEvent` | 既往史事件（多标签）| `Surgery` / `Trauma` / `BloodTransfusion` / `Vaccination` |
| `LifestyleFact` | 生活方式事实 | `type`, `value` |

### 核心关系类型

| 关系 | 说明 |
|---|---|
| `HAD_ENCOUNTER` | 患者 → 就诊 |
| `AT_HOSPITAL` | 就诊 → 医院 |
| `IN_DEPARTMENT` | 就诊 → 科室 |
| `TREATED_BY` | 就诊 → 医生 |
| `RECORDED_DIAGNOSIS` | 就诊 → 疾病 |
| `HAD_LAB_TEST` | 就诊 → 检验报告 |
| `HAS_ITEM` | 检验报告 → 检验项目（关系上存结果值） |
| `HAD_EXAMINATION` | 就诊 → 检查记录 |
| `HAS_FINDING` | 检查记录 → 疾病（发现） |
| `HAS_ALLERGY_TO` | 患者 → 过敏原 |
| `HAS_FAMILY_HISTORY` | 患者 → 疾病（家族史） |
| `HAS_LIFESTYLE_FACT` | 患者 → 生活方式事实 |
| `PARENT_OF` | 患者 → 患者（亲子关系） |
| `SPOUSE_OF` | 患者 ↔ 患者（配偶关系） |

---

## API 端点速查

所有端点均以 `/api/patients/<patient_id>/` 为前缀：

| 路径 | 方法 | 说明 |
|---|---|---|
| `dashboard` | GET | 患者仪表盘概览（基本信息 + 诊断 + 异常检验） |
| `encounters` | GET | 就诊记录列表（支持分页 `?page=1&limit=10`） |
| `history/medical` | GET | 既往医疗史（手术/外伤/输血/疫苗） |
| `history/personal` | GET | 生活方式/个人史 |
| `history/family` | GET | 家族史 |
| `allergies` | GET | 过敏史 |
| `marital_info` | GET | 婚育史 |
| `family-graph` | GET | 家族关系图谱（支持 `?depth=2`） |
| `/api/docs` | GET | 自动生成的接口清单页面 |

---

## 开发规范与约定

### 数据写入规范

1. **全部使用 `MERGE` 而非 `CREATE`**，以保证幂等性。`Encounter`、`Patient`、`Condition` 等核心节点均设有唯一性约束。
2. **批量写入优化**：`import_diagnoses_from_encounter` 采用两阶段写入——先批量 `UNWIND` 创建节点，再批量创建关系，避免高并发下的锁竞争。
3. **空值过滤**：各 `import_*` 函数内部均对关键字段进行空值检查，缺失时 `debug` 级别记录日志并跳过，不抛出异常。
4. **家族成员处理**：通过 `idType + idValue` 作为家族成员的唯一标识，支持"认领"机制——若 Neo4j 中已有通过家族关系预建的节点，ETL 会将 `patientId` 赋给它，实现无缝合并。

### 日志规范

- 使用 `setup_logger(name)` 获取 logger，禁止直接 `logging.getLogger()`。
- 每个模块使用独立的 logger 名称（`etl_patient_core`、`health_portrait`、`job_manager` 等）。
- 正常业务跳过用 `logger.debug()`，异常用 `logger.error()`，关键里程碑用 `logger.info()`。

### 配置规范

- 所有配置均在 `config/settings.py` 的 `Config` 类中集中管理。
- 不得在业务代码中硬编码 URI、密码、表名等敏感信息。
- 修改配置后，先运行 `python check_config.py` 验证有效性。

### 错误处理规范

- `HealthPortraitProcessor.process()` 捕获异常后**必须重新 `raise`**，以便 `JobManager` 将其计入错误队列并触发重试。
- `JobManager` 的错误重试上限由 `Config.RETRY_TIMES` 控制，重试间隔由 `Config.RETRY_DELAY` 控制。
- 若所有重试均失败，`etl_state.json` 的时间戳**不会更新**，下次运行会重新处理这些患者。

---

## 本地开发快速上手

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 检查环境配置
python check_config.py

# 3. 单次运行 ETL（使用本地 JSON 测试数据）
# 在 job_manager.py 中取消注释本地文件读取代码，注释 API 调用
python main.py

# 4. 启动 API 服务
python app.py
# 访问 http://localhost:5000/api/docs

# 5. 启动定时调度
python -c "from scheduler.scheduler import ETLScheduler; ETLScheduler().start(24)"
```

---

## AI Agent 任务指引

### 新增实体/关系类型
1. 在 `etl/core/etl_patient.py` 中新增 `import_xxx(tx, patient_id, data_list)` 函数。
2. 在 `import_patient_data_from_json` 的末尾调用新函数。
3. 在 `etl/core/__init__.py` 的 `__all__` 列表中导出。
4. 如需 API 查询，在 `app.py` 中新增对应端点，参照已有端点的 `@neo4j_session` 装饰器模式。

### 修改 Cypher 查询
- 所有 Cypher 均写在各 `import_*` 函数的局部变量 `query` 中，直接修改字符串即可。
- 修改后用 `files/test_patient.json` 作为输入运行单次测试验证。

### 新增 API 端点
```python
@app.route('/api/patients/<string:patient_id>/new_endpoint', methods=['GET'])
@neo4j_session
def get_new_endpoint(session, patient_id):
    """端点的中文描述（会显示在 /api/docs 页面）"""
    query = "MATCH (p:Patient {patientId: $patientId}) ..."
    results = session.execute_read(lambda tx: list(tx.run(query, patientId=patient_id)))
    return jsonify([serialize_record(r) for r in results])
```

### 调试单个患者
```python
# 临时修改 job_manager.py 的 _process_single 方法，
# 用本地 JSON 文件替代 API 调用：
with open("files/test_patient.json", 'r', encoding='utf-8') as f:
    patient_data = json.load(f)
patient_data = patient_data.get('data', patient_data)  # 兼容两种格式
```

---

## 已知限制与注意事项

- `MAX_WORKERS` 当前建议设为 `1`（串行），多线程并发写入 Neo4j 在当前图模型下可能引发死锁（`MERGE` 竞争）。
- `archive/` 目录中的文件为历史版本，请勿在新功能中引用。
- `app.py` 中已注释的部分端点（`findings`、`bodypart` 等）依赖图模型中尚未实现的节点类型，暂不可用。
- `etl_state.json` 存储的时间戳格式为 `2026-01-29 16:38:58 (Beijing)`，解析时需注意去除后缀。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cxblovedd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cxblovedd)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
