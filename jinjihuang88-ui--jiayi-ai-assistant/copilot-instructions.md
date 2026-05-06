## jiayi-dev

> 加移项目开发与部署规则，开发时务必遵守


# 加移 (jiayi.co) 开发规则

开发、改代码、部署时请先查看本规则并遵守。

## 一、输出与文件

- **涉及的文件**：始终使用**完整绝对路径**（例如 `c:\Users\User\jiayi-ai-assistant\src\app\api\rcic\auth\login\route.ts`）。
- **不需要出报告**：不单独写报告或说明文档，除非用户明确要求。
- **不说明错误出处，直接显示**：直接给出修改结果或正确写法，不解释错误出处；结果直接显示给用户。

## 二、部署前核对（本地与服务器数据库一致）

- **部署前必须核对**：本地 schema/迁移 与 **服务器（生产）数据库** 一致。
- 确认本地用的 schema 和服务器上的数据库一致，避免生产库和本地不一致导致：管理员能看到、但登录或别处出问题。
- 若需确认生产库，可让用户调用管理员接口或本地用相同 `DATABASE_URL` 跑脚本核对。

## 三、Prisma / TiDB 版本与数据库同步

- **Prisma 版本一致**：项目内 Prisma 与 package.json 保持一致。
- **TiDB 版本一致**：本地与服务器使用的数据库 TiDB 版本保持一致，避免兼容性问题。
- **数据库**：本项目使用 **TiDB Cloud（MySQL 协议）**，连接串环境变量为 **`DATABASE_URL`**。不要使用或依赖 `POSTGRES_*` 等旧变量。
- **Schema 与 MySQL**：TiDB/MySQL 不支持 Prisma 的 `mode: "insensitive"`；长文本使用 `@db.LongText` 或 `@db.Text`，避免 VARCHAR(191) 超长报错。
- **每次增加或修改功能**：都要与数据库同步——先执行 `prisma generate`，需要时执行 `prisma db push` 或 `prisma migrate`，保证**本地 schema 与数据库一致**，再构建与部署。

## 四、Vercel 部署

- 每次 Vercel 部署时会**自动同步数据库**（build 中含 prisma generate / db push）；只需**跟踪是否部署成功**。
- **部署成功后不需要生成报告**。
- 环境变量：生产环境只依赖 **`DATABASE_URL`**，不要依赖已废弃的 `POSTGRES_*`。

## 五、修改代码时的原则

- **首先确保不破坏已有正常功能**；改完后必要时做本地构建（`pnpm run build`）验证。
- **不能随便改变业务/产品流程**；若要改流程，**必须先问用户确认**后再改。

## 六、改代码或功能后的顺序

1. 先做**数据库同步**：`prisma generate`，需要时 `prisma db push` 或 `prisma migrate`，保证本地 schema 和数据库一致。
2. **部署前再次核对**：本地与服务器使用的数据库（TiDB）一致。
3. **先在本地给用户测试**：修改完毕在本地跑/测，不自行部署。
4. **等待用户通知后再部署**：用户明确说可以部署后，再执行 git push 触发 Vercel；不要未经用户同意自动部署。

## 七、移民顾问（RCIC）流程（不可改）

- **顺序固定**：注册 → **先验证邮箱** → 再管理员审核 → **审核通过后才能登录**。
- 登录接口必须校验：`emailVerified === true`、`approvalStatus === 'approved'`、`isActive === true`，否则拒绝登录并给出明确提示。
- 顾问与文案/操作员**同一登录入口**（如 `/rcic/login`），通过账号区分：先查持牌顾问（RCIC），再查团队成员；同一邮箱时**持牌顾问优先**。

---
> Source: [jinjihuang88-ui/jiayi-ai-assistant](https://github.com/jinjihuang88-ui/jiayi-ai-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
