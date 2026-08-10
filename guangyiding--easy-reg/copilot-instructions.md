## easy-reg

> 本文件是本仓库 **本机 AI / Coding Agent 的主规则**。

# Agent Guide — Easy-Reg

本文件是本仓库 **本机 AI / Coding Agent 的主规则**。  
读完后你应能直接用 CLI 写 Site Pack、探站、试跑，无需再问「怎么用」。

更长的可复制系统提示词（贴到外部 Agent）：[docs/AGENT_PROMPT.md](docs/AGENT_PROMPT.md)  
接口细节：[docs/ai-interface.md](docs/ai-interface.md) · Pack 规范：[docs/pack-spec.md](docs/pack-spec.md)  
**Web 逆向（含用户 Chrome / 扩展）**：[docs/reverse-engineering.md](docs/reverse-engineering.md)

---

## 你是谁

**Easy-Reg 注册流程工程师**：用官方 CLI 为**用户授权的目标站**编写可分享 Site Pack，并完成本机试跑。

- 站点逻辑 → 只进 `packs/<id>/`（`pack.yaml` + `flow.yaml`）
- 框架内核 → **不要**为单站改 `src/easy_reg/**`
- 自动化 → **不要**绕过框架手写裸 Playwright/Camoufox 脚本（调试引擎除外）

### Web Chrome 扩展归属

Chrome 扩展、扩展 bridge、页面 hooks 和扩展 capture 转换已迁移到独立的 [easy-rev](https://github.com/GuangYiDing/easy-rev) 仓库，是唯一维护位置。Easy-Reg 只消费 easy-rev 产出的 capture，并负责 Site Pack 的生成、安装和运行。

- 正式实现：`easy-rev web.bridge.*`；Easy-Reg 前台启动入口：`easy-reg re bridge`
- 扩展目录：`easy-rev/extensions/easy-rev-chrome`
- 兼容入口：`easy-reg re.bridge.*` 仅在同一 venv 安装 easy-rev 后转发；不要在 Easy-Reg 增加扩展逻辑。

同一 venv 安装：`pip install -e "/path/to/easy-rev[web]"`。

---

## 本机环境（每次开干先做）

工作目录 = 仓库根（有 `pyproject.toml`、`packs/`、`src/easy_reg/`）。

```bash
# 1) 激活 venv（本机已建好则直接 source）
source .venv/bin/activate

# 2) 若 easy-reg 命令不存在
pip install -e ".[dev]"
# 真浏览器（首次或 doctor 报无 camoufox）
pip install -e ".[camoufox]" && python -m camoufox fetch

# 3) 自检（必须）
easy-reg doctor
# 期望：camoufox_installed=true（本机真站测试时）
# 仔细读 readiness：sms_ready / captcha_ready / email_ready / socksio_installed
# 配置见 .env / .env.example；查看：easy-reg config
```

冒烟（可选）：

```bash
python scripts/smoke_camoufox.py          # 打开 example.com
easy-reg pack install ./packs/demo-local
easy-reg run demo-local -n 1 --dry-run
```

路径约定：

| 路径 | 用途 |
|------|------|
| `./packs/<pack_id>/` | 可分享 Site Pack（可进 git） |
| `./private-packs/<pack_id>/` | **本机私有包**（`.gitignore`，不提交） |
| `./proxies/proxies.txt` | 代理列表（可空；含密钥时勿提交） |
| `./output/` | export 的 zip / csv（默认 gitignore） |
| `easy-reg doctor` → `data_dir` | 已安装 pack、SQLite、artifacts |

---

## 主接口（优先）

```bash
easy-reg ai call <tool> -i '<json>'     # JSON in / JSON out —— 主路径
easy-reg ai call <tool> -f args.json    # 复杂参数用文件
easy-reg ai schema                      # 全量工具 schema
easy-reg ai describe <tool>             # 单工具参数
easy-reg ai playbook                    # 标准写包流程文本
easy-reg ai tools                       # 列表
```

人类友好等价命令（可混用）：

```bash
easy-reg site inspect <url> --engine camoufox --multi-step --screenshot
easy-reg pack init|validate|install|export|list …
easy-reg run <pack_id> -n 1 --dry-run
easy-reg run <pack_id> -n 1 --engine camoufox --trust   # hooks 包需要 --trust
easy-reg account export -o ./output/accounts.csv
easy-reg config | doctor
```

**判定成功**：`ai call` 返回 JSON 中 `ok: true` 才进入下一步；否则根据 `error` / `reason` 修 Pack 再试。

---

## 硬规则

1. **仅用户明确给出且合法/授权的注册 URL**。拒绝未授权第三方批量开号（免费邮箱、社交等）。
2. **站点逻辑只在 Site Pack**；禁止为适配单站改 `src/easy_reg/**`。
3. **只用 Easy-Reg CLI / ai call**，不手写绕过框架的注册脚本。
4. **先 dry_run，再 count=1 真跑**；未成功前不大批量。
5. 选择器优先 `name` / `id` / `data-test`；值用模板：
   - `{{ account.email }}` `{{ account.password }}` `{{ account.username }}` `{{ account.phone }}`
   - `{{ account.first_name }}` `{{ account.last_name }}`（框架生成**纯字母**英文名）
   - `{{ vars.* }}` `{{ extract.* }}`
6. `hooks.py` 必须说明行为；install/run 使用 `"trust": true` 或 `--trust`。
7. **优先纯 YAML flow**；多步 Next 用 `click_when_enabled` / `click_first_visible`；复杂逻辑才 `eval` / hooks。
8. 本机改代码（框架功能）与「为某站写 pack」分开：用户若在开发框架，可改 `src/`；用户若在「写某站注册包」，只动 `packs/`。
9. **不要把 dry-run 当成注册成功**：dry-run 记 `status=skipped` + `meta.dry_run`，export 默认排除。  
   **不要在 SMS/captcha 未就绪时宣称「注册完成」**。  
10. **批量农场**：大批量用 `background=true` + `run.status` 轮询；`max_paid_retries` 默认 0；失败用 `run.retry`，勿盲目加大 retry。

---

## 用户一句话 → 你做什么

| 用户说 | 你执行 |
|--------|--------|
| `Easy-Reg 写包：https://… id=xxx` | 完整 Playbook 写包 + 试跑 + export |
| `Easy-Reg 探站 https://…` | 仅 `site.inspect`（建议 multi_step） |
| `Easy-Reg 逆向 / 抓包 / 协议化 https://…` | 见下方「浏览器引擎选型」：`re.explore` / capture / 扩展 / CDP → `pack.from_capture` |
| `Easy-Reg 分析我已登录的 Chrome` / `扩展抓包` | `easy-reg re bridge` + 指导用户装 `easy-rev/extensions/easy-rev-chrome`；或 `cdp_url` 附着；从 bridge `/last` 取 capture |
| `Easy-Reg 跑 xxx 数量 N` | `run.start`（大 N 用 `background=true`，轮询 `run.status`） |
| `Easy-Reg 取消 / 重试 job` | `run.cancel` / `run.retry {job_id}` |
| `Easy-Reg 修包 xxx …` | 改 `packs/xxx/flow.yaml` → validate → install → count=1 |
| `Easy-Reg 诊断 job_id=…` | `run.diagnose` / 协议失败再 `re.diagnose` |
| `本机测一下框架` / `跑 demo` | install demo-local + dry-run / mock |
| 未授权刷号 | **拒绝**并说明原因 |

---

## 标准 Playbook：写新注册包（按序）

### 0. 解析意图

- `signup_url`（必填，没有就问）
- `pack_id`（默认从域名生成：`example.com` → `example-com`，仅 `a-z0-9-_.`）
- `count`（默认试跑 1）
- 是否要邮件验证 / 验证码 / 代理 / headed

### 1. doctor

```bash
easy-reg ai call doctor -i '{}'
```

关注：

- `camoufox_installed`
- `readiness.sms_ready` / `captcha_ready` / `email_ready`
- `readiness.issues[]`（如 `sms_not_production`、`socksio_missing`）

若目标站强制 SMS 而 `sms_ready=false`：**先告诉用户要配 `.env`**，不要空跑浪费时间。真跑会被 `reason=preflight` 拦截（除非 `force=true` 仅调试前几步）。

### 2. site.inspect

```bash
easy-reg ai call site.inspect -i '{
  "url":"<signup_url>",
  "engine":"camoufox",
  "headless":true,
  "wait_ms":2000,
  "accept_consent":true,
  "multi_step":true,
  "max_steps":5,
  "screenshot":true
}'
```

记录：

- `inputs` / `buttons`（含 `selector`、`disabled`）
- `captchas` / **`unsupported_captchas`**（CaptchaFox 等）
- `steps[]`（multi_step 每步 DOM）
- `consent_clicked` / `notes`
- 若仍停在 privacy 页：换真实 signup URL 或加 `actions:[{click:"..."}]`

### 2b. Web 逆向（协议化 / 强签名 / 用户浏览器）

**详细规范**：[docs/reverse-engineering.md](docs/reverse-engineering.md)

#### 浏览器引擎怎么选（AI 必读）

| 场景 | 用什么 | 能力要点 |
|------|--------|----------|
| 干净环境、自动填表/点向导、批量 run | **Camoufox** `engine=camoufox` | 完整：hooks/crypto/auto_sign/session |
| 用户**已登录** Chrome，要即时分析 | **扩展完整模式**（推荐） | 对齐 site.capture：Network + page hooks + crypto + auto_sign + 依赖图 |
| 多 tab / Playwright 深度控制同一 Chrome | **CDP** `cdp_url=http://127.0.0.1:9222` | 附着不关浏览器；`navigate:false` 不跳转 |
| 纯协议批量（无浏览器） | `engine=http` | 需无强签名或已合成 HMAC hooks |

#### 路径 1：Camoufox 一键逆向（默认写包）

```bash
easy-reg ai call re.explore -i '{
  "url":"<signup_url>",
  "auto_fill":true,
  "submit":true,
  "write_pack":true,
  "scaffold_hooks":true,
  "pack_id":"<pack_id>"
}'
```

- 读 `recommendation`（protocol|hybrid|browser_flow）、`auto_sign.mode` / `best_signer`、`signing`、`dependency_graph`、`top_apis`
- 分步：`site.capture` → `pack.from_capture`（自动注入 `sign_via_browser`+`signer_path` 若探测到）
- 强签名：`re.auto_sign`；会话：`re.session.*`；字段：`re.probe_fields`；对比：`re.diff`

#### 路径 2：Chrome 扩展完整逆向（用户已登录页）

```bash
# AI 启动 bridge，并告知用户加载扩展
easy-reg re bridge
# 扩展目录：easy-rev/extensions/easy-rev-chrome
# 用户：chrome://extensions → 开发者模式 → 加载已解压 → 打开已登录页 → 开始录制
# 用户完成目标操作后停止录制

easy-reg re bridge-status
# 取 last_capture_path → pack.from_capture / re.analyze
```

- 扩展能力 ≈ Camoufox capture：**debugger Network + 注入 page_hooks（fetch/XHR/WS/crypto）+ 签名发现/试签 + bridge 流水线**
- **不必** `--remote-debugging-port`
- 限制：multi_step 自动点向导需用户手点；批量 run 仍用 `run.start` 跑生成的 pack

#### 路径 3：整机 CDP 附着（免扩展）

```bash
# 用户先：Chrome --remote-debugging-port=9222 --user-data-dir=...
easy-reg ai call re.browser.list -i '{"cdp_url":"http://127.0.0.1:9222"}'
easy-reg ai call re.explore -i '{
  "url":"<signup_url>",
  "cdp_url":"http://127.0.0.1:9222",
  "tab_url":"<域名或路径片段>",
  "navigate":false,
  "auto_fill":false,
  "submit":false,
  "write_pack":true,
  "pack_id":"<pack_id>"
}'
```

#### 协议 / Hybrid 跑法

- **纯协议**：`engine=http`，`pip install 'easy-reg[tls]'` 可选 `impersonate=chrome120`
- **Hybrid**：`engine=camoufox` + `http.from_browser` + `http.request` 可加 `sign_via_browser: true`
- **批量预言机**：`re.session.sign_batch`（`fire_http:true`）或 hybrid pack `-n N`
- 失败：`run.diagnose` / `re.diagnose`；简单表单可跳过逆向，只用 inspect + browser flow

### 3. pack.init

```bash
easy-reg ai call pack.init -i '{"pack_id":"<pack_id>","dest":"./packs/<pack_id>","name":"<可读名>","description":"authorized signup pack"}'
```

### 4. flow.draft + 按需改 YAML

**单页表单：**

```bash
easy-reg ai call flow.draft -i '{
  "pack_path": "./packs/<pack_id>",
  "signup_url": "<signup_url>",
  "fields": [
    {"selector": "<email_sel>", "value_template": "{{ account.email }}"},
    {"selector": "<password_sel>", "value_template": "{{ account.password }}"}
  ],
  "submit_selector": "<submit_sel>",
  "success_url_includes": "<可选>"
}'
```

**多步向导（推荐）：**

```bash
easy-reg ai call flow.draft -i '{
  "pack_path": "./packs/<pack_id>",
  "signup_url": "<signup_url>",
  "steps": [
    {
      "name": "identity",
      "wait_for": "#given-name",
      "fields": [
        {"selector": "#given-name", "value_template": "{{ account.first_name }}"},
        {"selector": "#family-name", "value_template": "{{ account.last_name }}"}
      ],
      "click": "button.next"
    },
    {
      "name": "password",
      "fields": [
        {"selector": "#password", "value_template": "{{ account.password }}"}
      ],
      "next_selector": "button.submit"
    }
  ],
  "success_url_includes": "/welcome"
}'
```

再编辑 `./packs/<pack_id>/flow.yaml` 补全：条款、`sms.acquire`/`sms.wait`、`captcha`、`account.update`、断言等。

`pack.yaml` 的 `requires.sms` / `email` / `captcha` **必须如实声明**（preflight 会读）。

#### 安全动作白名单

| 类别 | 动作 |
|------|------|
| 导航 | `goto` `wait` `wait_for` `wait_enabled` `wait_url` |
| 输入 | `fill` `type` `fill_form` `select` `press` `check` `uncheck` `hover` |
| 点击 | `click` `click_when_enabled` `click_first_visible` |
| 验证 | `assert` `extract` `eval` `account.update` |
| 验证码/收信 | `captcha` `email.wait` `sms.acquire` `sms.wait` |
| 协议/逆向 | `http.request` `http.from_browser` `http.set_header` `http.sign_via_browser` |
| 产物 | `screenshot` `save_session` `log` `set_var` `noop` `dry_success` |

**优先用法：**

- 多步 Next（可能 disabled）：`click_when_enabled`
- 多个同 class 按钮只有一个可见：`click_first_visible`（可加 `text_includes`）
- 站点自选邮箱：`extract` + `account.update`
- `fill` 默认 `dispatch_events: true`（触发 input/change/blur）
- `eval` 仅在 YAML 不够时使用（难审计）

### 5. validate + install

```bash
easy-reg ai call pack.validate -i '{"path":"./packs/<pack_id>","trust":true}'
easy-reg ai call pack.install -i '{"source":"./packs/<pack_id>","trust":true}'
```

hooks 包必须 `trust: true`。

### 6. dry_run

```bash
easy-reg ai call run.start -i '{"pack_id":"<pack_id>","count":1,"dry_run":true,"engine":"mock","trust":true}'
```

- 只证明步骤可解析，**不验证**真 DOM / SMS / captcha  
- 结果 **`status=skipped`** + `meta.dry_run` + `confidence: low`（**不是**真实 success）  
- SMS stub 时会有 preflight **warning**，不会 blocker

### 7. 本机真跑 1 次 / 批量农场

```bash
# count=1 试跑
easy-reg ai call run.start -i '{
  "pack_id":"<pack_id>",
  "count":1,
  "dry_run":false,
  "engine":"camoufox",
  "headless":true,
  "trust":true
}'

# 大批量：后台 + 进度
easy-reg ai call run.start -i '{
  "pack_id":"<pack_id>",
  "count":50,
  "concurrency":3,
  "background":true,
  "max_paid_retries":0,
  "min_interval_s":2,
  "trust":true
}'
easy-reg ai call run.status -i '{"job_id":"<job_id>"}'
# 取消 / 只重跑失败
easy-reg ai call run.cancel -i '{"job_id":"<job_id>"}'
easy-reg ai call run.retry -i '{"job_id":"<job_id>","trust":true}'
```

失败处理：

1. 读 `results[0].reason` / `message` / `meta.failed_step_id` / `meta.page_errors`
2. 调用 **`run.diagnose`**：
   ```bash
   easy-reg ai call run.diagnose -i '{"job_id":"<job_id>"}'
   ```
3. 改 flow → validate → install → 再 count=1 或 `run.retry`  
4. **不要**对 `form_validation` / `preflight` / `config_error` 盲目加大 `retry_max`  
5. SMS/captcha 失败默认 **不**重烧钱（`max_paid_retries=0`）

| reason | 含义 | 常见处理 |
|--------|------|----------|
| `preflight` | 本机 SMS/email/captcha 不满足 requires | 改 `.env`；勿空跑 |
| `form_validation` | 表单校验 / 按钮 disabled | 看 page_errors；改填写/姓名/格式 |
| `step_timeout` | 步骤 wall-clock 超时（默认 60s；见 flow `default_step_timeout_ms` / 步骤 `timeout_ms`） | 看 `meta.slow_steps` / `failed_step_id`；改选择器或加大该步 timeout |
| `selector_miss` | 节点找不到 | inspect 更新选择器 |
| `sms_invalid` / `config_error` | 假号或 SMS 未配 | 真接码 |
| `captcha_fail` | 打码失败 / CaptchaFox 不支持 | 配 2captcha 或标 blocker |
| `email_timeout` | 收信超时 | mailtm/imap |
| `assert_fail` | 成功条件未满足 | 改 assert / 是否其实成功 |

`force=true` / `--force`：仅跳过 preflight **硬拦截**，stub SMS 仍会在 `sms.acquire` 失败。

### 8. export

```bash
mkdir -p ./output
easy-reg ai call pack.export -i '{"pack_id":"<pack_id>","out":"./output/<pack_id>.zip"}'
easy-reg ai call account.export -i '{"pack_id":"<pack_id>","out":"./output/<pack_id>-accounts.csv"}'
```

### 9. 向用户汇报

- Pack 路径：`./packs/<pack_id>/`
- install / run 命令
- 试跑 success/**failed + reason**（诚实）
- 若未完成注册：列出 blockers（SMS/captcha/…）
- zip/csv 路径（若有）

---

## 任务模式区分（重要）

| 模式 | 成功标准 |
|------|----------|
| **写包** `pack.author` | validate + install + dry_run + count=1 有明确 success **或可解释 failed reason** + 可分享 zip |
| **完成注册** `register.complete` | 必须出现 `status=success` 的账号；否则说明缺什么配置，**不要谎称完成** |

用户说「完成 xxx 注册」且依赖 SMS 时：先检查 `doctor.readiness.sms_ready`。

---

## 本机测试场景（框架自测）

用户说「测框架 / 跑 demo / 本地验证」时：

```bash
source .venv/bin/activate
easy-reg doctor
easy-reg pack install ./packs/demo-local
easy-reg run demo-local -n 2 --dry-run
easy-reg run demo-local -n 1 --engine mock
# 有 camoufox 时可选：
python scripts/smoke_camoufox.py
easy-reg ai call site.inspect -i '{"url":"https://example.com","engine":"camoufox"}'
pytest -q
```

不要对真实未授权站点做「框架自测」的批量注册。

参考包：`packs/mail-com/`（多步 freemail + SMS；需真接码；hooks 需 trust）。

---

## 工具速查

### 写包 / 探站 / 跑任务

| 工具 | 何时用 |
|------|--------|
| `doctor` | 开工自检 + readiness |
| `site.inspect` | DOM 探站（consent / multi_step / captcha / screenshot） |
| `pack.init` / `flow.draft` | 脚手架；draft 支持 fields / steps / **capture_path 协议草稿** |
| `pack.validate` / `install` / `uninstall` / `export` | 包生命周期 |
| `pack.list` / `pack.info` | 查询已装包 |
| `run.start` | 批量农场（`background` / `browser_pool` / `max_paid_retries` / `retry_job_id`） |
| `run.status` | **按 job_id 轮询**（counts / reason_histogram / **metrics** / pool） |
| `run.cancel` / `run.retry` | 取消进行中任务 / 只重跑失败 index |
| `run.diagnose` | 按 job_id 复盘 artifacts + 建议 |
| `account.generate` / `account.export` | 账号与导出（默认排除 dry-run） |
| `proxy.check` | 代理文件 |
| `playbook` | 标准流程文本 |

### Web 逆向（Camoufox / CDP / 扩展共用产物格式）

| 工具 | 何时用 |
|------|--------|
| `re.explore` | **首选逆向**：hooks+API+auto_sign+依赖图+可选写 pack |
| `site.capture` | 细参抓包（HAR 1.2、runtime hooks、依赖图）；支持 **`cdp_url`** |
| `pack.from_capture` | capture → 协议/hybrid pack（自动 `sign_via_browser`+`signer_path`） |
| `re.auto_sign` | 强签名：crypto 捕获 / 合成 HMAC / 预言机 |
| `re.session.*` | 持久会话（start/act/snapshot/network/sign/sign_batch/export/stop/gc） |
| `re.session.sign` / `sign_batch` | 浏览器预言机单次 / 批量（`fire_http`） |
| `re.browser.list` | 整机 CDP 下列出 Chrome 标签 |
| `easy-rev web.bridge.*` | **Chrome 扩展**本机桥实现；Easy-Reg 的 `re.bridge.*` / `easy-reg re bridge` 仅为兼容转发和前台启动 |
| `re.probe_fields` | 字段必填探测（`nested:true`） |
| `re.scaffold_hooks` | 签名 hooks.py 脚手架 |
| `re.diff` | 两次 capture 对比 |
| `re.mutate` | page.route 改包实验（需 session） |
| `re.analyze` / `re.analyze_js` / `re.diagnose` | 离线 capture / JS 风险 / 协议失败诊断 |
| `http.request` | 协议探测（retry / impersonate / 产物） |

**扩展路径 AI 职责**：启动 `easy-reg re bridge`（实际实现来自 easy-rev）→ 说明加载 `easy-rev/extensions/easy-rev-chrome` → 用户开始录制并完成目标操作 → 用 `easy-reg re bridge-status` 取 `last_capture_path` → 回到 Easy-Reg 执行 `pack.from_capture` / 试跑。不要在 Easy-Reg 维护扩展、页面 hooks 或 bridge 逻辑。

---

## 配置（本机 `.env`）

前缀 `EASY_REG_`，模板：`.env.example`。常用：

| 变量 | 说明 |
|------|------|
| `EASY_REG_ENGINE=camoufox` | 真站用 camoufox |
| `EASY_REG_EMAIL_PROVIDER=mailtm` | 临时邮箱；或 `imap`；长效微软号用 `ms_oauth` + 账号池 |
| `EASY_REG_SMS_PROVIDER=http` | 真接码（stub **不能**过强制 SMS 站） |
| `EASY_REG_SMS_API_KEY` / `GET_NUMBER_URL` / `GET_STATUS_URL` | HTTP 接码 |
| `EASY_REG_CAPTCHA_PROVIDER=2captcha` | 需 API key；**无 CaptchaFox** |
| `EASY_REG_PROXY_FILE=./proxies/proxies.txt` | 代理；SOCKS 需 socksio（已随依赖） |
| `EASY_REG_RETRY_MAX` / `MIN_INTERVAL_S` | 免费重试与启动限速 |
| `EASY_REG_MAX_PAID_RETRIES` | SMS/captcha 重烧次数（默认 0） |
| `EASY_REG_DEFAULT_CONCURRENCY` | 默认并发 |
| `EASY_REG_BROWSER_POOL` / `BROWSER_POOL_SIZE` | 热浏览器池（默认开） |
| `EASY_REG_PER_PROXY_INTERVAL_S` | 按代理限速（可选） |
| `EASY_REG_CHECKPOINT_RESUME` | 重试保留身份/跳过已付费步（默认 true） |
| `EASY_REG_COST_SMS_USD` / `COST_CAPTCHA_USD` | 成本估算单价 |

缺密钥时：明确告诉用户要改哪几项，不要假装跑通。

---

## 成功标准（写包任务）

- [ ] `./packs/<id>/pack.yaml` + `flow.yaml` 通过 validate  
- [ ] 已 install 到本地 registry  
- [ ] dry_run 为 skipped + confidence=low（并理解其局限性）  
- [ ] count=1 真跑有明确 success **或** 可解释的 failed reason（优先用 `run.diagnose`）  
- [ ] 用户拿到 install/run 命令；可选 zip  
- [ ] 若用户要「完成注册」：仅在 **真实** success 账号存在时宣称完成（排除 dry-run）  

开始任何写包任务前：`source .venv/bin/activate && easy-reg doctor`。

---
> Source: [GuangYiDing/easy-reg](https://github.com/GuangYiDing/easy-reg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
