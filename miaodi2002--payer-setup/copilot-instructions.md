## payer-setup

> 记录对本仓库做 Payer 部署时**实际踩过、并且会再次踩**的坑。新会话读完这份文档应该能避免重复验证和试错。

# Payer Setup — Claude 操作手册

记录对本仓库做 Payer 部署时**实际踩过、并且会再次踩**的坑。新会话读完这份文档应该能避免重复验证和试错。

## 项目速览

- 入口脚本：`aws-payer-automation/scripts/deploy-single.sh`（单模块部署）与 `deploy.sh`（全量）
- 8 个模块：OU+SCP → BillingConductor → CUR Pro forma → CUR RISP → Athena → Account Auto Management → CloudFront Monitoring → IAM Users
- CUR API 只在 `us-east-1`；所有 CFN 栈默认走 us-east-1
- Payer 账户需要已经开通 AWS Billing Conductor

## 模板目录布局（重要）

```
aws-payer-automation/templates/
├── 01-ou-scp/..08-iam-users/      ← deploy-single.sh 实际读取这里
├── current/                        ← 另一份"当前稳定版"快照
└── versions/v1/..v1.5/             ← 历史版本归档
```

**坑**：`current/` **不是 symlink**，是独立目录。修模板时只改一个会让另一个保持旧版，下次部署仍复现老 bug。  
**规则**：修 `05-athena-setup/athena_setup_fixed.yaml`、`07-cloudfront-monitoring/oam-link-stackset.yaml` 等模板时，**必须同时改 `templates/<module>/` 和 `templates/current/<module>/` 两份**（或先改 current/，再 `cp` 覆盖）。

## 部署前必查清单

1. **凭证区域**：`export AWS_DEFAULT_REGION=us-east-1`（CUR 只支持此区域）
2. **账户身份**：`aws sts get-caller-identity` 确认 Payer Master Account ID
3. **已有 OU/SCP**：某些历史 Payer 用 `auto-ou` + `organizations-SCP` 栈名（非 `payer-*`），不要重部 Module 1 否则会 SCP/OU 重复
4. **BillingGroup 已存在？**：`aws billingconductor list-billing-groups`；若 `Bills` 存在就跳过 Module 2

## 各模块已知 Bug 与处理

### Module 2 — BillingConductor
- Lambda 代码调用了**不存在的 `boto3.client('billingconductor').get_billing_group`**（应为 `list_billing_groups`）
- **后果**：第一次成功创建 BillingGroup 后，"verify" 分支抛异常；fallback 路径第二次 create 撞到 "Cannot associate accounts, which are already associated with billing group" → CFN 进入 ROLLBACK_COMPLETE
- **关键**：CFN rollback **不会删除**已创建的 BillingGroup 和 Bills 账户（它们是 custom resource 创建的外部资源）
- **应对**：看到 Module 2 ROLLBACK，先 `aws billingconductor list-billing-groups` —— 如果 `Bills` 已 ACTIVE，直接删掉失败栈，拿 BillingGroup ARN 继续 Module 3，**不要重部 Module 2**
- `deploy_module3` 需要 `--billing-group-arn arn:aws:billingconductor::<PAYER_ID>:billinggroup/<BILLS_ACCOUNT_ID>`

### Module 5 — Athena Setup
- `deploy-single.sh` 必须指向 `athena_setup_fixed.yaml`（v1.5，含 IAM 传播等待 + Schedule）
- 基础版 `athena_setup.yaml` 有两个致命问题：
  1. 无 IAM 角色传播等待 → `InvalidInputException: Service is unable to assume provided role`
  2. 无 `Schedule` → Crawler 建好但永远不自动跑，Glue DB 没有表
- 修复版创建的 role 名 `AWSCURCrawlerRole-<ACCOUNT_ID>`（无 UUID 后缀）；旧版是带 8 位 UUID 后缀
- **验收**：部署后 `aws glue get-crawler --name AWSCURCrawler-<ID>` 必须看到 `Schedule.ScheduleExpression = "cron(0 2 * * ? *)"`
- **Crawler 名大小写**：Pro forma = `AWSCURCrawler-<ID>`；RISP = `AWSCURCrawler-**RISP**-<ID>`（大写）。所有 verify 脚本对比时用大写

### Module 7 — CloudFront Monitoring + OAM Link StackSet
service-managed StackSet 需要**两处**分别启用，缺一不可：
```bash
aws organizations enable-aws-service-access \
  --service-principal member.org.stacksets.cloudformation.amazonaws.com
aws cloudformation activate-organizations-access --region us-east-1
```
只启用第一个会报错：`"You must enable organizations access to operate a service managed stack set"`。`deploy-single.sh:enable_stacksets_trusted_access()` 现在同时做这两件事。

`oam-link-stackset.yaml` 对 `AWS::Oam::Link` 有两个 schema 陷阱：
1. **Tags 是 Map，不是 List**：
   ```yaml
   # 错（普通 CFN 资源写法）
   Tags:
     - Key: Name
       Value: foo
   # 对（Oam::Link 特有）
   Tags:
     Name: foo
     Purpose: bar
   ```
   错误写法导致 `"#/Tags: expected type: JSONObject, found: JSONArray"`
2. **`LinkConfiguration.MetricConfiguration.Filter` 的 JSON-object 格式 OAM 不接受**，会报模糊错误 `"Invalid request provided: AWS::Oam::Link"`。目前模板**去掉整个 LinkConfiguration**，共享所有 CloudWatch metrics；Alert Lambda 在查询时已按 `Namespace=AWS/CloudFront` 过滤，不产生实际影响

**StackSet 部署目标：Root OU（不是 Normal OU）**。原 `deploy.sh` 用 Normal OU ID，但成员账户常常还在 Root（Module 6 的 EventBridge 规则只对 *新* 账户生效，不移动历史账户）。Root OU 可覆盖所有成员账户。

### Module 8 — IAM Users
**`deploy-single.sh` 没有 `case 8`**；直接 `aws cloudformation create-stack` 部署：
```bash
aws cloudformation create-stack \
  --stack-name "payer-iam-users-$(date +%s)" \
  --template-body file://templates/08-iam-users/iam_users_init.yaml \
  --parameters ParameterKey=PayerName,ParameterValue=<PayerName> \
  --capabilities CAPABILITY_NAMED_IAM
```

## 验证清单（下次部署完成后跑）

```bash
ACCOUNT_ID=<payer-account-id>
# 1. Crawler Schedule（必须非 None）
aws glue get-crawler --name AWSCURCrawler-$ACCOUNT_ID \
  --query 'Crawler.Schedule.ScheduleExpression' --output text
aws glue get-crawler --name AWSCURCrawler-RISP-$ACCOUNT_ID \
  --query 'Crawler.Schedule.ScheduleExpression' --output text
# 2. OAM Sink 的 Links（必须等于成员账户数）
SINK_ARN=$(aws oam list-sinks --query 'Items[0].Arn' --output text)
aws oam list-attached-links --sink-identifier "$SINK_ARN" \
  --query 'length(Items)' --output text
# 3. BillingGroup Size（应 >= 1）
aws billingconductor list-billing-groups \
  --query 'BillingGroups[?Name==`Bills`].Size' --output text
# 4. CUR Export 必须有 2-3 个（Pro forma + RISP + 可选 bill-for-m-console）
aws cur describe-report-definitions --query 'length(ReportDefinitions)'
# 5. IAM 用户必须含 cost_explorer 和 ReadOnly_system
aws iam list-users --query 'Users[*].UserName' --output text
```

## 下次可能还会失败的点

- **首次 CUR 数据延迟**：CUR 报告第一次生成需要 ~24h。部署当天 Crawler 跑了也没表是正常；次日 02:00 UTC 的 cron 自动补上。验证时遇到 0 表别慌，先 `aws s3 ls s3://bip-cur-<ID>/daily/ --recursive` 确认 bucket 里有 parquet 文件
- **临时凭证过期**：长时间部署（StackSet 等）可能跑超 1 小时。凭证失效症状是 `InvalidClientTokenId`。续签凭证后可无缝续跑，StackSet 操作不会因此回退
- **已有账户不在 Normal OU**：Module 6 的 EventBridge 只抓 *新* 账户创建/邀请事件。历史手工邀请的成员账户仍在 Root，要么手动 `aws organizations move-account`，要么 StackSet 部署到 Root OU（当前 `deploy-single.sh` 已改为此策略）

## 最近会话修过的文件（参考提交）

- `aws-payer-automation/scripts/deploy-single.sh`
  - Module 5 改用 `athena_setup_fixed.yaml`
  - Module 7 加入 OAM Link StackSet 部署 + 两次 trusted-access 启用
- `aws-payer-automation/templates/05-athena-setup/athena_setup_fixed.yaml`（与 `current/` 同步）
  - Schedule 从隐式 `create_crawler` 参数改为显式 `update_crawler_schedule` + 结果校验
- `aws-payer-automation/templates/07-cloudfront-monitoring/oam-link-stackset.yaml`（与 `current/` 同步）
  - Tags 从 list 改为 map
  - 去掉 `LinkConfiguration` 整段
- `payer-deployments/scripts/verify-elite-new17.sh`
  - RISP Crawler 名称 `risp` → `RISP`

---
> Source: [miaodi2002/payer-setup](https://github.com/miaodi2002/payer-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
