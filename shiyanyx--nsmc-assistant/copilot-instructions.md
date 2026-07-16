## nsmc-assistant

> Grade-query desktop app for NSMC (川北医学院) students. Scrapes the school's academic management system (教务系统) by logging in as the student and parsing HTML grade tables. Packaged as a Tauri desktop app (Windows only).

# NSMC-Assistant (川北医助手) — CLAUDE.md

## Overview

Grade-query desktop app for NSMC (川北医学院) students. Scrapes the school's academic management system (教务系统) by logging in as the student and parsing HTML grade tables. Packaged as a Tauri desktop app (Windows only).

**Primary flow:** User enters student ID + password → frontend calls localhost:5000 backend → backend logs into `jiaowu3.nsmc.edu.cn` → scrapes grade tables → returns JSON to frontend.

## Architecture

```
┌─────────────────────────────────────────────────┐
│  Tauri Desktop App (frontend/)                   │
│  React 18 + Vite + FluentUI React v8/v9         │
│  Auto-launches backend.exe from resources/      │
│  Calls http://localhost:5000/api/*              │
└──────────────┬──────────────────────────────────┘
               │ HTTP (localhost:5000)
┌──────────────▼──────────────────────────────────┐
│  Backend (one of two options):                   │
│  • Flask (app.py) — primary Python backend       │
│  • Actix-web (backend_rust/) — Rust rewrite      │
│  Both: login proxy + HTML scraper for            │
│  jiaowu3.nsmc.edu.cn                            │
└─────────────────────────────────────────────────┘
```

Two backends exist with identical API surface. The Tauri app bundles `backend_rust/target/release/backend_rust.exe` as `resources/backend.exe`. The Flask version (`app.py`) is also in the repo and works standalone.

## Project Structure

```
NSMC-Assistant/
├── app.py                    # Flask backend (Python)
├── score_fetcher_http.py     # Grade scraping + evaluation module (imported by app.py)
├── xg2_fetcher.py            # xg2 学工系统登录 + 去向登记爬虫
├── backend_rust/             # Rust backend (Actix-web)
│   ├── Cargo.toml
│   └── src/main.rs           # All logic in single file
├── frontend/                 # Tauri + React frontend
│   ├── index.html            # Entry HTML
│   ├── vite.config.js
│   ├── package.json          # React 18, FluentUI, Tauri API
│   ├── src/
│   │   ├── main.jsx          # React root + FluentProvider
│   │   ├── App.jsx           # Login screen + main layout + backend launcher
│   │   ├── index.css         # Minimal base styles
│   │   └── components/
│   │       ├── ScoreQuery.jsx       # Grade display table + filtering
│   │       ├── EvaluationQuery.jsx  # Teaching evaluation UI
│   │       ├── LeaveRegistration.jsx # 去向登记 form (dev-tagged)
│   │       └── areaData.js          # 省市区数据（去向登记用）
│   └── src-tauri/            # Tauri Rust shell
│       ├── Cargo.toml
│       ├── tauri.conf.json   # Window config, bundle settings
│       ├── capabilities/default.json
│       └── src/
│           ├── main.rs       # Windows subsystem entry
│           └── lib.rs        # Tauri setup: launch backend.exe, cleanup on close
├── main_page.html            # Scraped 教务系统菜单页（开发参考）
├── xg2_diag_resp.html        # xg2 学工系统响应抓取样本（开发参考）
├── xg2_resp_Custom.html       # xg2 学工系统响应抓取样本（开发参考）
├── xg2_resp_Pycryptodome.html # xg2 学工系统响应抓取样本（开发参考）
└── README.md
```

## API Endpoints (both backends)

### POST /api/login
```json
// Request
{ "username": "学号", "password": "密码" }
// Response (success)
{ "success": true, "data": { "username": "...", "name": "姓名" }, "message": "登录成功" }
// Response (failure)
{ "success": false, "message": "登录失败，请检查学号和密码" }
```

### POST /api/score
```json
// Request
{ "username": "学号", "password": "密码", "name": "可选姓名", "term": "可选学期" }
// Response (success)
{ "success": true, "data": { "username": "...", "name": "...", "scores": [{...}] } }
```

### POST /api/evaluation/list
```json
// Request
{ "username": "学号", "password": "密码" }
```

### POST /api/evaluation/submit
```json
// Request
{ "username": "学号", "password": "密码", "teacher": {教师对象}, "do_submit": true }
```

### POST /api/xg2/login
```json
// Request
{ "username": "学工号", "password": "密码" }
```

### POST /api/xg2/edit-form
```json
// Request
{ "username": "学工号" }
```

### POST /api/xg2/submit
```json
// Request
{ "username": "学工号", "form_fields": {表单字段对象} }
```

## Key Implementation Details

### Authentication
- Target: `https://jiaowu3.nsmc.edu.cn/jsxsd/`
- Login POST to `xk/LoginToXk` with `encoded=base64(user)%%%base64(pass)`
- Success check: response URL contains `xsMain` or response text contains `个人中心`
- All requests use `verify=False` (ignore SSL cert errors)

### Grade Scraping
- Term list page: `/jsxsd/kscj/cjcx_query` → `select#kksja` options
- Grade table page: `/jsxsd/kscj/cjcx_list?kksj={term}`
- Scrape target: `table#dataList` → first row = headers, rest = data rows
- Column mapping in `ScoreQuery.jsx` lines 133-144: 开课学期, 课程编号, 课程名称, 成绩, 学分, 总学时, 绩点, 考核方式, 考试性质, 课程属性

### 去向登记（LeaveRegistration / xg2 学工系统）
- Target: `xg2.nsmc.edu.cn`（学工系统）
- Uses RSA encryption via jsencrypt (JS) and PyCryptodome (Python): password encrypted with server modulus before POST
- Login POST to `xsd/xsdsql.aspx` with encrypted credentials; success checked via redirected URL
- Scrapes 节假日列表 → 编辑表单 → 提交登记
- Frontend component: `LeaveRegistration.jsx` (marked with "开发中" badge)
- Backend module: `xg2_fetcher.py` (Python), `main.rs` partial (Rust)
- Frontend login credentials stored separately in `localStorage` as `xg2_saved_login`

### Frontend State Management
- Login credentials persisted via `localStorage` (`currentUser`, `savedLogin`)
- Backend launched via Tauri shell plugin (`Command.sidecar` / `Command.create`)
- Backend health check: POST a test login to `localhost:5000` with dummy credentials
- Port fallback: 5000→5001→5002→5003→5004

### Tauri Backend Lifecycle
- `lib.rs` `setup()`: launches `resources/backend.exe` (from Rust `backend_rust`), hides console window via `CREATE_NO_WINDOW`
- `on_window_event(Destroyed)`: kills backend with `taskkill /F /PID` on Windows

### Dev vs Production
- `tauri.conf.json`: `devUrl: http://localhost:3000`, `frontendDist: ../dist`
- In Tauri env, frontend imports `@tauri-apps/api` dynamically; in browser dev, these are skipped
- Vite config excludes `@tauri-apps/api` from bundling — Tauri provides them at runtime

## Running the App

### Flask backend (dev)
```bash
python app.py  # starts on port 5000-5004
python score_fetcher_http.py  # CLI test mode
```

### Rust backend (dev)
```bash
cd backend_rust && cargo run
```

### Frontend (dev)
```bash
cd frontend && npm install && npm run dev  # Vite on :5173
```

### Full Tauri build
```bash
cd frontend && npm install && npm run tauri build
# Output: frontend/src-tauri/target/release/app.exe
# Requires: backend_rust compiled and copied to frontend/src-tauri/resources/backend.exe
```

## Dependencies

### Python (app.py, score_fetcher_http.py)
- flask, flask-cors, requests, beautifulsoup4, pandas, urllib3

### Rust (backend_rust/)
- actix-web 4, actix-cors 0.7, reqwest 0.12, scraper 0.19, serde/serde_json, base64 0.21, tokio

### Frontend (package.json)
- react 18, @fluentui/react (v8) + @fluentui/react-components (v9) — both versions used
- @tauri-apps/api 2.x, axios (listed but unused)
- vite 5, @vitejs/plugin-react

### Tauri shell (frontend/src-tauri/)
- tauri 2.10, tauri-plugin-shell 2.0, tauri-plugin-log 2

## Agent skills

### Issue tracker

Issues tracked on GitHub at `shiyanYX/NSMC-Assistant`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at repo root. See `docs/agents/domain.md`.

---
> Source: [shiyanYX/NSMC-Assistant](https://github.com/shiyanYX/NSMC-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
