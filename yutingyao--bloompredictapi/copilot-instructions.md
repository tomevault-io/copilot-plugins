## bloompredictapi

> - **依赖安装**: `pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/`

# 蓝藻预测系统开发规范

## 🚀 快速开始
- **依赖安装**: `pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/`
- **启动服务**: `source venv/bin/activate && python start_server.py`
- **详细说明**: 先阅读 README.md

## 📁 目录规范
- **文档**: 放在 `docs/` 目录，文件名需注明日期 (格式: YYYYMMDD)
- **测试**: 放在 `test/` 目录
- **日志**: 自动生成在 `logs/` 目录

## 🛠️ 开发工具
### MCP工具优先级 (必须使用)
1. **代码分析**: 优先使用 `mcp_serena_*` 工具进行代码搜索、符号查找
2. **项目理解**: 使用 `mcp_serena_get_symbols_overview` 了解文件结构
3. **代码修改**: 使用 `mcp_serena_replace_symbol_body` 等进行精确编辑
4. **项目记忆**: 使用 `mcp_serena_write_memory` 记录重要发现

### 代码质量
- **类型提示**: 必须添加完整的类型注解
- **异步处理**: 正确使用 `async/await`
- **异常处理**: 完备的错误处理机制
- **代码规范**: 严格遵循 PEP 8

## 🎯 核心约束
### 模型管理
- **支持模型**: LSTM、GRU-D、TCN、XGBoost
- **支持站点**: 胥湖心、锡东水厂、平台山、tuoshan、lanshanzui、五里湖心
- **加载策略**: 懒加载 + 内存缓存

### API设计
- **版本管理**: V1(8000)、V2(8001)、V3(8002)
- **数据校验**: 使用 Pydantic 严格校验
- **响应格式**: 统一 `{success, data, error}` 结构
- **文档生成**: 自动生成 OpenAPI/Swagger 文档

## ✅ 验证流程
1. **代码修改后**: 必须运行相关测试脚本
2. **模型相关**: 验证所有4种模型在6个站点的功能
3. **API变更**: 确保所有版本的接口正常
4. **性能检查**: 监控内存使用和响应时间

---
> Source: [YutingYao/BloomPredictAPI](https://github.com/YutingYao/BloomPredictAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
