## zsd-aidemo

> 系统中所有金额必须使用 `Money` 值对象封装，严禁使用原生 `decimal` 进行裸运算。


# 财务业务规则 (Finance Business Rules)

<precision_rules>
  **重要：金额精度铁律**
  系统中所有金额必须使用 `Money` 值对象封装，严禁使用原生 `decimal` 进行裸运算。
  - **含税金额 (TaxIncluded)**: 保留 **2位** 小数 (Math.Round(val, 2))。
  - **不含税/成本 (TaxExcluded)**: 保留 **10位** 小数 (Math.Round(val, 10)) 以减少累计误差。
</precision_rules>

<business_logic>
  <write_off_logic>
    **冲销 (Write-off) 规则**
    只有满足以下 **所有** 条件时，正负单据才可冲销：
    1. 同一项目 (Project)
    2. 同一业务单元 (BusinessUnit)
    3. 同一供应商 (Agent)
    4. 同一公司主体 (Company)
    
    **优先级**: 预付账期 > 入库账期 > 销售账期 > 回款账期
  </write_off_logic>

  <claim_logic>
    **认款 (Claim) 规则**
    - **互斥性**: 同一认款单只能认款到一种类型（发票 OR 订单 OR 初始应收）。
    - **盘点限制**: 财务盘点期间，相关项目禁止任何认款操作。
  </claim_logic>
</business_logic>

<glossary>
  - **寄售 (Consignment)**: 货物所有权归上游，使用结算。
  - **经销 (Distribution)**: 买断货物，所有权归我方。
  - **总额法**: 按全额确认收入。
  - **净额法**: 按差价确认收入。
</glossary>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abindef) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
