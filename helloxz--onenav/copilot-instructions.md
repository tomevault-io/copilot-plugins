## onenav

> - 全程使用中文输出，包括分析、进度说明、变更说明、提交信息建议；不要默认切回英文。

# AGENTS

- 全程使用中文输出，包括分析、进度说明、变更说明、提交信息建议；不要默认切回英文。

## 入口与分发

- 真实入口是根目录 `index.php`，不要从 README 猜流程。
- `index.php` 启动顺序固定：先检查 `data/config.php`，不存在就进入 `controller/init.php`；再检查 `data/onenav.db3`，不存在时从 `db/onenav.simple.db3` 复制；然后 `require ./data/config.php`；最后按 `?c=` 分发到 `controller/*.php`。
- 未传 `c` 时进入 `controller/index.php`；`?c=api`、`?c=admin`、`?c=login`、`?c=mobile` 等都直接映射到 `controller/<name>.php`。
- `index.php` 会拒绝包含 `./` 的控制器参数；改路由时不要破坏这层基于文件名的动态分发。
- `.htaccess` 只做两条重要伪静态：`/click/<id>` -> `index.php?c=click&id=<id>`，`/api/<method>` -> `index.php?c=api&method=<method>`。

## 关键目录

- `controller/` 放页面/入口控制器，真正的 API 业务主要在 `class/Api.php`，不要只改 `controller/api.php` 的参数转发层。
- `functions/helper.php` 放登录态、主题枚举、移动端跳转等共享函数；前台首页和后台都依赖它。
- 后台管理的大部分前端交互、表格渲染和 API 调用集中在 `templates/admin/static/embed.js`；改后台管理行为时要同时检查这个文件。
- 内置主题在 `templates/`，用户自定义主题走 `data/templates/`；代码里默认先找内置目录，再回退到 `data/templates/`。
- `db/onenav.simple.db3` 是初始化种子库，`data/onenav.db3` 是运行中的真实数据。

## 运行与状态文件

- `data/config.php` 和 `data/onenav.db3` 是本地实例状态文件，且被 `.gitignore` 忽略；除非用户明确要求，不要覆盖、删除、重建这两个文件。
- 安装流程依赖 `config.simple.php` 模板生成 `data/config.php`；涉及初始化逻辑时，同时检查 `controller/init.php` 和 `config.simple.php`。
- `controller/init.php` 的可执行约束比 README 更可信：初始化会检查 `pdo_sqlite`、`openssl`、`zlib`、`curl`，并拒绝二级目录安装。

## 前台与主题流程

- `controller/index.php` 会先查 `on_options.s_site` 和分类/链接，再决定主题；不要只看模板目录名判断首页行为。
- 未配置主题时默认 `default2`；已配置时按 UA 在 `s_themes` 中区分 `pc_theme` 和 `mobile_theme`。
- `?theme=` 只允许切到 `get_all_themes()` 返回的主题；这个函数会排除 `admin`、`mobile`、`universal`。
- 主题配置文件兼容两套命名：优先 `config.json`，其次 `info.json`；内置主题和 `data/templates/` 都要一起兼容。

## 认证与 API 约束

- 登录态不是 PHP session，而是 `key` cookie；`functions/helper.php::is_login()` 会把它和 `USER + ENCRYPTED_PASSWORD + onenav + User-Agent` 的 MD5 比较。
- `class/Api.php::auth()` 允许两种 API 鉴权：`token=md5(USER + SecretKey)` 或请求头 `X-Token`；没传 token 时会退回 cookie 登录态。
- 后台和移动后台都依赖同一套 cookie 登录判断；改鉴权时要一起看 `controller/admin.php`、`controller/mobile.php`、`controller/login.php`。

## 验证方式

- 仓库里没有可验证的 `composer.json`、`package.json`、CI workflow、lint/test 配置；不要臆造 `composer test`、`npm run build` 之类命令。
- 对 PHP 改动，最小可执行验证是对改过的文件运行 `php -l <file>`。
- 对路由或接口改动，优先按真实入口验证：`/index.php?c=...` 或重写后的 `/api/...`、`/click/...`。

---
> Source: [helloxz/onenav](https://github.com/helloxz/onenav) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
