## health-skill

> 医疗垂域安全智能助手。规则兜底 + LLM推理 + 降级回退。


# Health Agent Skill

医疗垂域安全智能助手。混合决策架构：规则兜底 + LLM 推理 + 降级回退。

## 快速使用
```
@health-agent /health-init user_id=demo age=24 sex=female medical=髋关节手术
@health-agent /health-query user_id=demo "我想练深蹲"
```

## 标准指令（5 个）

### /health-init
初始化用户健康档案。

语法：
```
/health-init user_id=demo age=24 sex=female weight=55 medical=髋关节手术
```

### /health-query
智能健康咨询（规则+LLM）。

语法：
```
/health-query user_id=demo "我想练臀桥"
```

### /health-stats
查看使用统计。

### /health-benchmark
快速评测（10 个 cases）。

### /health-add-rule
添加自定义规则。

语法：
```
/health-add-rule user_id=demo condition=孕妇 tag=禁忌 action=仰卧起坐
```

## 返回示例
```
{"risk_level":"dangerous","need_hitl":true,"advice":"这是禁忌动作，建议停止并咨询医生。"}
```

## 安装
```
curl -sSL https://raw.githubusercontent.com/hazelian0619/health-skill/main/install.sh | bash
```

## License
MIT

---
> Source: [hazelian0619/health-skill](https://github.com/hazelian0619/health-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
