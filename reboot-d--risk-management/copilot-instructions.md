## risk-management

> Building a secure data management system with frontend and backend components, focusing on risk data handling and visualization.

# Secure Data Management System Development Guide

## Project Overview
Building a secure data management system with frontend and backend components, focusing on risk data handling and visualization.

## Development Phases

### Backend Development
- Phase 1: Basic Infrastructure (Current)
  * TypeScript + Express setup
  * Tencent Cloud MySQL connection
  * Security middleware
  * Base configurations

- Phase 2: Core API Development
  * Authentication endpoints
  * Data management endpoints
  * Logging system

- Phase 3: Security & Processing
  * Data validation
  * Risk processing
  * Audit system

### Frontend Development
- Phase 1: UI Framework
  * React + TypeScript
  * Routing setup
  * State management
  * Component library

- Phase 2: Core Pages
  * Auth pages
  * Data management
  * Dashboard views

- Phase 3: Advanced Features
  * Real-time updates
  * Data visualization
  * User preferences

## Technical Requirements
- Backend: Node.js, TypeScript, Express, Tencent Cloud MySQL
- Frontend: React, TypeScript, UI Framework
- Security: JWT, encryption, access control
- Quality: TypeScript types, error handling, logging

## Development Standards
1. Use TypeScript strictly
2. Implement security best practices
3. Follow proper error handling
4. Use environment variables for sensitive data
5. Maintain clear documentation 

gz-cdb-bh19rij3.sql.tencentcdb.com

username: root
password: AB8Vyft3koyn9x**h4PAw

协议端口: TCP:3306
源地址: 您的公网 IP 或 0.0.0.0/0 (允许所有外部 IP)


只需要这个功能: 前端设计界面获得对方提供的下面字段, 后段数据库进行储存, 并且可以随时提取和查看数据
一、基本交易信息字段
1.mc_create_trade_ip: 商户端创建订单的外网IP地址（IPv4/IPv6）
2.mcCreateTradeTime: 交易创单时间
3.mcCreateTradeChannelType: 交易商品分类（实物/虚拟）
4.mcCreateTradeChannel: 交易商品分类渠道或内容（如账户充值、虚拟币购买、卡券购买等）
5.tradeChannelRiskLevel: 交易商品分类渠道的风险等级（high, mid, low, rel）
6.isFundFreezeBiz: 是否资金保证金业务（Y/N）
二、外部账户（被充值账户）信息字段
1.extraAccountRegTime: 外部账户注册时间
2.extraAccountName: 外部账户姓名（部分脱敏）
3.extraAccountCertno: 外部账户证件号（可加密）
4.extraAccountCertnoLastSix: 外部账户证件号后6位（明文）
5.extraAccountPhone: 外部账户手机号（建议先实名认证）
6.extraAccountPhoneLastFour: 外部账户手机号后4位（明文）
7.extraAccountPhoneRegTime: 外部账户手机号账户绑定时间
8.loginDeviceQuantity: 外部账户近30天登录设备数量
9.alipayUserCustomerId: 外部账户关联支付宝买家ID
10.desensitizedUid: 外部账户ID（保持唯一性）
11.extraAccountRiskLevel: 外部账户风险等级（high, mid, low, rel）
12.extraAccountBusinessLevel: 外部账户业务等级（1, 2, 3等）
13.extraAccountBusinessLevelReason: 外部账户业务等级判定原因
三、被充值外部账号信息字段
1.chargedCardNumber: 被充值外部账号ID（保持唯一性）
2.chargedCardNumberRiskLevel: 被充值外部账号ID风险等级（high, mid, low, rel）
四、二级商户信息字段
1.extraMerchantId or secondaryMerchantNo: 订单收款外部商户ID（保持唯一性）
2.extraMerchantRiskLevel: 外部商户风险等级（high, mid, low, rel）
五、其他信息字段
1.extraCreateTradeRiskLevel: 交易风险等级（high, mid, low, rel）
2.extraCreateTradeControlMethod: 交易管控商户建议方式（交易拦截、交易校验、交易放行）
3.LoanType: 借款类型（如消费贷、现金贷、企业贷款等）
4.Instalments: 分期数（如1, 3, 6, 9, 12等）
5.RepaymentTimes: 已还期数（需与Instalments字段组合使用）
六、脱敏与数据安全处理
涉及个人敏感数据的字段应采用单向哈希函数进行脱敏，以确保数据的不可还原性，仅用于风险管控，不用于其他用途。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Reboot-D) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
