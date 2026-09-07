## stax

> 本项目**没有独立 dev 云环境**:本地开发直连生产云环境 `cloud1-d7gykoaktfc01fbf0`。

# Stax 开发契约(所有 AI Agent 与人类开发者必读)

本项目**没有独立 dev 云环境**:本地开发直连生产云环境 `cloud1-d7gykoaktfc01fbf0`。
云函数一经部署立即对**所有线上用户**生效;线上正式版前端的源码曾经丢失,只能靠契约兼容保护它。
以下规则均来自真实线上事故,违反任何一条都可能直接打挂线上。

## 云函数契约红线(cloudfunctions/)

1. **绝不按枚举名称拒绝请求**。不得新增 `INVALID_MODE` 这类"白名单外即拒"校验;
   未知的 mode/枚举值要么按默认分支处理,要么按**载荷形状**推断意图(参考 settleGame)。
   事故:INVALID_MODE 白名单拒掉了线上前端的费用分摊调用。
2. **绝不删除/改名任何已写入数据库或已返回给前端的字段**。只能新增。
   废弃字段必须继续双写至少一个发布周期(参考 settledCount/checkedOutCount)。
   事故:删 settledCount 导致线上房间无法收局;rankings 丢 winRate 导致胜率全 0。
3. **绝不收紧已有入参校验**。新校验只能作用于新参数;老前端能通过的调用,新函数必须继续通过。
   事故:whoami 对不合格昵称整体拒绝,险些让老用户启动即废(应忽略该参数而非拒绝)。
4. 新错误码必须同步在前端加文案映射,且前端**永不裸弹原始错误码**(fallback 统一"操作失败,请稍后重试")。
5. 改动云函数后必须先跑 `npm run check` 再部署;**先部署云函数,再上传前端**。

## 多 Agent 协作

6. 开工前先看 `git status`:树上可能有其他 Agent 的未提交改动,**不得回退/覆盖他人改动**,只做定点修改。
7. cloud-mock.js 与云函数是镜像关系:改任何云函数行为,必须同步改 mock 并保持测试全绿。
8. 完成一个独立单元就提交(小步 commit),commit 信息写清改了什么契约。
9. 数据准确 > 体验 > 性能。涉及结算/积分/胜率的改动,必须有测试覆盖。

## 快速自检(部署云函数前)

```bash
npm run check   # 单元 + Mock E2E + 合规,必须全绿
git diff <上次部署的commit> -- cloudfunctions/<fn>/ | grep -E "^-" # 检查删了什么
```
被删的行里若有:返回字段、数据库写入字段、放行逻辑 → 停下来,按红线 1-3 重新设计。

---
> Source: [disheng5/Stax](https://github.com/disheng5/Stax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
