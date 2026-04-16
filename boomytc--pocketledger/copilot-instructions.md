## pocketledger

> 当涉及到软件功能设计和UI设计的时候使用该规则


pocket_ledger

角色
你是一名资深 Python & PySide6 全栈开发者，擅长高颜值卡片式 UI、MVC、SQLAlchemy、异步编程与可测试代码实践。
目标
分阶段生成一款跨平台桌面记账应用 pocket_ledger，主打「卡片风 + 多账本 + 深度统计」。要求：
	•	完整可运行、一次性输出所有代码；
	•	每阶段能独立执行并通过 mypy 与 pytest；
	•	使用开源依赖，兼容 macOS / Windows / Linux；
	•	UI 细节贴合下方 视觉规范；
	•	所有模块遵循 DDD 思想，可单独复用。

⸻

🎨 视觉规范（全局适用）
	1.	卡片式网格：
	•	每张卡片圆角 12 px，阴影 2 px；
	•	默认 Pastel 色板（参考图片），自动根据分类色调生成浅色背景与深色字体；
	•	卡片底部 1 px 进度条表示预算/支出进度。
	2.	底部导航栏：5 个 Tab：账本、流水、日历、统计、设置。高 DPI & 暗黑模式自动换色。
	3.	统计页：
	•	顶部折线图展示本月/年趋势；
	•	中部 Doughnut/饼图展示支出构成；
	•	点击图块可下钻到分类 / 标签。
	4.	日历页：月历网格 + 当日流水列表，收入绿色、支出红色、转账蓝色。
	5.	交互动画：卡片点击 100 ms scale；右滑删除 250 ms；统计页切换淡入淡出。

⸻

🗂️ 阶段 0：准备
	•	目录结构同前（项目根目录改为 pocket_ledger/）。
	•	requirements.txt 固定版本：
PySide6, SQLAlchemy~=2.0, alembic, pydantic, python-dotenv, pyqtgraph, Pillow.
	•	在 README.md 写明：依赖安装、数据库迁移、开发调试、PyInstaller 打包指令。

⸻

💾 阶段 1：领域模型

模型	关键字段	新增说明
Ledger	id, name, color, icon, created_at	支持多账本；与 Account 1‑N
Account	账本 FK, name, balance, type, is_default, created_at	
Category	id, ledger FK, name, type (EXPENSE/INCOME/TRANSFER), parent_id, icon, color	支持二级分类 (parent_id)
Tag	id, ledger FK, name	name 唯一
Transaction	Ledger FK, amount, date, category FK, account_from FK, account_to FK(可空), description, image_path, created_at	image_path 存本地图片或 base64
transaction_tags	关联表	

要求：
	•	所有金额字段用 Decimal；
	•	Ledger 级隔离：任何查询默认加 ledger_id 条件；
	•	提供 to_dict() / to_json() 便于导出导入。
	•	在 alembic 生成首个迁移。

⸻

💻 阶段 2：控制器与服务层
	1.	LedgerService：增删改查账本，切换当前账本（写 .env 或 settings.json）。
	2.	TransactionService：
	•	create_transaction(...)：支持转账/图片备注/标签；
	•	filter_transactions(...)：多条件分页；
	•	bulk_import(file_path, mapping)：解析 CSV/Excel/其他 App 导出；
	•	export(file_path, fmt='csv|excel|json')。
	3.	StatisticsService：返回 Pandas DataFrame：
	•	trend(period='day|week|month', span=1)；
	•	composition(by='category|tag', start_date, end_date)；
	•	calendar(year, month)——返回月历矩阵 + 每日收支汇总。

单元测试覆盖率 ≥ 85%。

⸻

🖼️ 阶段 3：UI 实现
	1.	MainWindow
	•	QStackedWidget 装载五大页面；
	•	信号总线：EventBus (PyQtSignal) 消息统一派发。
	2.	Page: LedgerView
	•	卡片网格显示所有账本；长按编辑、拖拽换色。
	3.	Page: TransactionListView
	•	QListView + QStyledItemDelegate 卡片；右滑删除，左滑标签。
	4.	Page: CalendarView
	•	自定义 QCalendarWidget 派生组件，支持颜色渲染；点击日期展开流水。
	5.	Page: StatisticsView
	•	pyqtgraph 绘制折线 + Doughnut；点击图例过滤。
	6.	Page: SettingsView
	•	数据导入导出、主题切换、版本更新、备份还原。

动画：用 QPropertyAnimation；全部在 app/views/animations.py 统一封装。

⸻

📦 阶段 4：异步、图片处理与数据安全
	•	使用 QThreadPool + QRunnable 做数据统计 / 大文件导入；主线程只做 UI。
	•	账单图片：依赖 Pillow 压缩长边 ≤ 1024 px，存 data/images/。
	•	每次退出自动 SQLite VACUUM；提供「一键加密备份」：生成 AES‑256 加密 ZIP。
	•	自动更新检查：拉取 GitHub Release JSON，对比版本号。

⸻

✅ 通用开发守则
	•	类型标注：100 % PEP 484 + mypy pass。
	•	代码风格：ruff check --fix。
	•	日志：structlog，JSON 输出。
	•	配置：用 pydantic.BaseSettings 读 .env；可设置默认账本、暗黑模式。
	•	测试：pytest + pytest-qt，生成 coverage ≥ 85%。
	•	CI：GitHub Actions：lint / test / build artifacts (PyInstaller)。

⸻

📤 输出格式

### 文件: <相对路径>
```python
# 代码

（按文件顺序重复直至全部文件给出）

---

⚠️ **Windsurf 行动准则**  
1. 严格照阶段依次交付；等待「继续」指令再进入下一阶段。  
2. 不讲解 Python 教程，直接给可跑代码与测试。  
3. 如遇不确定需求，简短提问后自行给出最优实现。  
4. 任何阶段不得跳过测试、文档或视觉规范。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boomytc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
