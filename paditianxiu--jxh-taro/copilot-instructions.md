## jxh-taro

> 这是一个面向校园服务场景的微信小程序，当前基于 `Taro 4 + React 18 + TypeScript + webpack5 + Tailwind CSS v4 + weapp-tailwindcss` 构建。业务方向包括校园登录、首页、课表/日程、通知、交流与后续校园服务能力。

# AGENTS.md

## 项目定位

这是一个面向校园服务场景的微信小程序，当前基于 `Taro 4 + React 18 + TypeScript + webpack5 + Tailwind CSS v4 + weapp-tailwindcss` 构建。业务方向包括校园登录、首页、课表/日程、通知、交流与后续校园服务能力。

本文件适用于整个仓库。

## 必须遵守

- 始终使用简体中文回复。
- 先读代码再修改，基于事实判断，不凭猜测改动。
- 若用户没有明确要求，不要执行 `git commit`、`git push`、创建/切换分支等 Git 操作。
- 不直接编辑 `dist/` 下的构建产物；源代码改动应落在 `src/`、`config/`、`types/` 或项目配置文件中。
- 不提交、输出或硬编码校园账号、学号、密码、Cookie、Token、AppSecret、接口密钥等敏感信息。
- 登录、个人信息、课表、通知等涉及学生隐私的数据，不允许在 `console.log` 中明文输出。

## 危险操作确认

执行以下操作前必须获得用户明确确认：

- 删除文件或目录、批量移动/重命名、覆盖大量文件。
- `git commit`、`git push`、`git reset --hard`。
- 修改全局环境变量、系统配置、权限配置。
- 数据库删除、结构变更、批量更新。
- 调用生产环境 API、发送敏感数据。
- 全局安装/卸载包、升级核心依赖。

确认格式：

```text
⚠️ 危险操作检测！
操作类型：[具体操作]
影响范围：[详细说明]
风险评估：[潜在后果]

请确认是否继续？[需要明确的"是"、"确认"、"继续"]
```

## 项目结构

- `src/app.ts`：Taro 应用入口。
- `src/app.config.ts`：小程序全局页面与窗口配置。
- `src/app.css`：全局样式入口，当前引入 Tailwind CSS。
- `src/pages/index/`：登录页，当前品牌为“江小航”。
- `src/pages/home/`：首页/日程页雏形。
- `src/assets/`：图片、字体图标等静态资源。
- `types/`：全局类型声明。
- `config/`：Taro 构建配置，包含 webpack、路径别名、weapp-tailwindcss 配置。
- `dist/`：微信小程序构建输出目录，只作为产物查看，不作为源码维护入口。

## 常用命令

```bash
pnpm install
pnpm dev:weapp
pnpm build:weapp
pnpm build:h5
pnpm open
```

说明：

- 微信小程序开发优先使用 `pnpm dev:weapp`。
- 发布或验证微信端产物使用 `pnpm build:weapp`。
- `pnpm open` 依赖微信开发者工具 CLI。
- 依赖安装后会运行 `weapp-tw patch`，不要随意移除 `postinstall`。

## 技术约束

- 使用 Taro 组件与 API：优先从 `@tarojs/components`、`@tarojs/taro` 引入，不使用浏览器 DOM API 直接操作页面。
- 页面组件使用函数组件，状态逻辑优先使用 React Hooks。
- 路径别名使用 `@/*` 指向 `src/*`。
- 小程序页面必须在 `src/app.config.ts` 的 `pages` 中注册。
- 页面级配置放在同目录 `index.config.ts`。
- 当前项目使用 `navigationStyle: "custom"`，涉及顶部安全区和导航栏时必须考虑微信小程序设备差异。
- 样式以 Tailwind 原子类为主；复杂、复用或平台兼容性要求高的样式再放入页面 CSS。
- 不引入新的 UI 框架、状态管理库或请求库，除非当前需求确实需要且用户确认。

## 代码风格

- 遵循 `.editorconfig`：2 空格缩进、UTF-8、文件末尾换行。
- TypeScript 保持现有配置约束，避免引入未使用变量和未使用参数。
- 代码注释语言与现有代码保持一致，当前项目以中文注释为主。
- 命名要表达业务含义，例如 `studentId`、`scheduleItems`、`campusNotice`，避免无意义缩写。
- 保持 KISS：优先直接、清晰的实现。
- 保持 YAGNI：只实现当前明确需要的能力，不预留复杂框架。
- 保持 DRY：重复 UI 或业务逻辑出现稳定模式后，再抽取组件、Hook 或工具函数。
- 保持 SOLID：页面负责组合，业务请求、数据转换、复用 UI 应逐步拆分到明确职责的模块。

## 校园服务业务规则

- 学号、密码、手机号、身份证号、课表、成绩、宿舍、定位等都按敏感信息处理。
- 登录表单应使用受控或清晰可追踪的状态管理，不在日志中打印明文密码。
- 密码输入必须使用安全输入类型；涉及保存凭据时必须先确认安全策略。
- 课表、考试、通知等时间数据应明确时区和日期来源，避免硬编码长期存在的模拟数据。
- 面向学生的文案要简洁明确，避免营销化表达。
- 校园服务功能优先保证可用性、稳定性和隐私边界，再考虑视觉强化。

## 前端实现要求

- 首屏应直接进入可用功能，不做无意义落地页。
- 微信小程序 UI 必须优先适配移动端窄屏，检查文本换行、按钮高度、底部安全区和滚动区域。
- 图标优先复用现有 `src/assets/fonts/icon.css` 或项目已有资源。
- 不用纯表情符号作为正式功能图标；原型阶段可以存在，但进入功能实现时应替换为稳定图标资源。
- 固定底部导航、抽屉、弹层等组件必须避免遮挡内容，滚动容器要留出底部空间。
- 涉及图片资源时放入 `src/assets/`，通过模块导入使用。

## 文件修改边界

- 新增页面时：
  - 在 `src/pages/<page>/` 下创建页面文件。
  - 补充 `index.config.ts`。
  - 在 `src/app.config.ts` 注册页面。
- 新增共享组件时放在 `src/components/`。
- 新增业务请求封装时放在 `src/services/`。
- 新增通用工具函数时放在 `src/utils/`。
- 新增业务类型时优先放在靠近业务的文件中；跨页面复用后再放到 `src/types/` 或 `types/`。
- 避免为一次性逻辑创建过深目录。

## 验证要求

完成代码变更后，根据影响范围选择验证：

```bash
pnpm build:weapp
pnpm build:h5
```

若只修改文档，可不运行构建，但需要说明未运行的原因。若构建失败，必须报告失败命令、关键错误和已确认的影响范围。

## 代理工作流程

1. 读取相关文件，确认当前实现和约束。
2. 给出简短执行思路；若涉及危险操作，先请求确认。
3. 小步修改，保持变更聚焦。
4. 执行必要验证。
5. 总结改动、验证结果、剩余风险。

---
> Source: [paditianxiu/jxh_taro](https://github.com/paditianxiu/jxh_taro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
