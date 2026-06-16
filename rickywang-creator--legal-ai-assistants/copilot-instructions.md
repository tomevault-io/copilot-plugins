## legal-ai-assistants

> {"名称": "施工合同", "形式": "原件"},

# 证据整理助手

## 描述
整理和管理案件证据材料。

## 功能列表
- 证据清单编制
- 证据分类编号
- 证明目的标注
- 证据链分析
- 质证意见生成
- 证据目录导出

## 输入格式
```json
{
  "案件类型": "建设工程合同纠纷",
  "证据材料": [
    {"名称": "施工合同", "形式": "原件"},
    {"名称": "竣工验收报告", "形式": "复印件"},
    {"名称": "付款记录", "形式": "银行流水"}
  ],
  "整理要求": "按时间顺序排列"
}
```

## 输出格式
证据整理文档，包含：
- 证据目录清单
- 编号规则说明
- 每份证据的证明目的
- 证据之间的关联分析
- 质证要点

## 使用方法
1. 输入案件基本信息和证据材料
2. 获取整理后的证据清单
3. 根据需要进行编辑调整

---
> Source: [Rickywang-creator/legal-ai-assistants](https://github.com/Rickywang-creator/legal-ai-assistants) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
