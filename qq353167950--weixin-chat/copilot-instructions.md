## weixin-chat

> `gui/index.html` 是**构建产物**，由 `gui/build_redesign.py` 把下列源文件拼成单个自包含页面（离线打进 exe）：

# 项目开发约定

## 前端产物由构建生成，不入库

`gui/index.html` 是**构建产物**，由 `gui/build_redesign.py` 把下列源文件拼成单个自包含页面（离线打进 exe）：

- `gui/_redesign.css` —— 视觉系统
- `gui/_redesign_body.html` —— 页面结构（无 `<html>`/`<head>`）
- `gui/_logic_*.js` —— 应用逻辑，按职责拆分（纯 JS，不带 `<script>` 外壳）

产物**不入库**（在 `.gitignore` 里），无需手动同步：

- **源码运行**：`gui_app.py` 启动前自动重建（`build(only_if_stale=True)`，仅源文件更新时写盘）
- **本地打包**：`build_exe.bat` 在 PyInstaller 之前显式构建
- **CI**：装完依赖即构建，然后才跑自测与打包

需要手动看产物时跑 `python gui/build_redesign.py`。

> 历史教训：产物曾入库并靠 CI 哈希比对强制同步，v2.3.1 因忘记重建而丢掉整个安装包。现在产物由构建唯一产生，不存在「两个真值需要保持一致」这件事。

### `_logic_*.js` 的拼接顺序不可随意调整

所有 JS 拼进同一个 `<script>` 块、共享同一个全局作用域。顶层 `const`/`let` 不提升，靠后文件里的常量若被靠前文件的顶层立即执行代码读到会 TDZ 报错。顺序定义在 `build_redesign.py` 的 `SCRIPTS` 列表里，新增文件放在 `_logic_boot.js` **之前**（启动引导必须排最后）。

这三类致命问题由 `build_redesign.render()` 在构建时直接拦住（缺文件、误加 `<script>` 外壳、跨文件 TDZ 引用），构建失败即中止，不会产出坏页面 —— 所以 CI 跑构建就等于跑检查。`scripts/selftest.py` 的「前端源文件能拼出完整页面」再覆盖一层，含真阴性用例（函数体/回调体/字符串里的同名引用不许误报）。

`PUB_LIMITS`（`_logic_publish.js`）的键与 `_redesign_body.html` 的元素 id 是隐式契约：`updateCharCount` 直接取 `#cnt-<后缀>`，事件绑定又遍历这些键去 `addEventListener`。少一个元素页面加载即 TypeError，构建和 TDZ 扫描都看不见 —— 自测里有专项检查。

## 发版流程

版本号在 `scripts/version.py` 的 `__version__`，**必须与 git tag 一致**（CI 会校验，不一致则中止）。

发版前在本地完整复现 CI 的检查：

```bash
# 1. 版本号与目标 tag 一致
python -c "import sys; sys.path.insert(0,'scripts'); from version import __version__; print(__version__)"

# 2. 离线冒烟自测（务必用项目 .venv，系统 Python 常缺 flask/dotenv 导致假失败）
./.venv/Scripts/python.exe scripts/selftest.py
```

两项全过后：更新 `RELEASE_NOTES.md`（每版一段，CI 自动提取对应段落作为 Release 说明）→ 提交 → `git tag -a vX.Y.Z -m "..."` → 推送 main 与 tag。

标签一律用 `git tag -a` 附注标签，与历史版本保持一致。

## 验证

`scripts/selftest.py` 是离线冒烟自测，覆盖排版、封面、预览、去 AI 味、发布流程、前端构建等，不调任何外部 API。改动后必跑。

只跑关心的用例：`python scripts/selftest.py -k 排版`（按名字子串过滤，`-k` 可给多个）。

## 依赖注入优先于 ambient 读取

配置统一由 `scripts/app_config.py` 在边界处解析一次，往下**传值**，不要在函数体里 `os.getenv`。
业务模块的外部能力（LLM 调用、生图、微信客户端、日志）一律作为参数注入，便于离线测试替身。

公开函数用 `cfg: X | None = None` + `cfg = cfg or 解析()` 的形式（`publish_to_draft`、`make_cover`、`deai_article` 都是这个形状）：调用方可以不传，测试可以注入。私有函数直接收必需参数，不再自己解析。

两个别在迁移中丢掉的细节：

- **区间钳制**。`env_int` 的 `minimum`/`maximum` 不是装饰 —— 设置页填 `ARTICLE_MIN_CHARS=999999` 会让提示词要求写一百万字。加新配置项时想清楚它的合理区间。
- **「未配置」与「配了某个值」是两回事**。`ImageConfig.provider_explicit` 就为此存在：封面必须有图，所以 provider 缺省填 openai；而正文配图在用户没明确选生图服务商时不该悄悄调 API 花钱。缺省值会吃掉「没配」这个信息，需要区分时显式记下来。

## 决策记录

`docs/adr/` 放不该被反复重提的决策。若某次评审/重构建议被否掉、且理由是未来的人无从得知的（例如「这个 CLI 我在用」），记一条 ADR，别让下一轮评审重新提一遍。

---
> Source: [qq353167950/weixin_chat](https://github.com/qq353167950/weixin_chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
