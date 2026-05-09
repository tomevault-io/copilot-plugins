## project-init

> `myquery` is a **CLI + Web hybrid assistant** for interacting with databases using natural language.


## 🧠 Project Rules — `myquery`

### 📛 Project Name

**myquery**

---

### 🧩 Description

`myquery` is a **CLI + Web hybrid assistant** for interacting with databases using natural language.
It turns plain English into SQL, executes queries, and visualizes insights — all powered by **Python**, **LangChain**, **OpenAI**, and **MCP (Model Context Protocol)** on port **7766**.

Think of it as **“ChatGPT + DataGrip + TablePlus”** running in your terminal or browser.

---

### ⚙️ Tech Stack

* **Language:** Python
* **Core Libraries:** LangChain, OpenAI, SQLAlchemy, Typer, Rich
* **Visualization:** Plotly / Matplotlib / Altair
* **Web UI:** FastAPI + React (or Svelte)
* **Databases Supported:** PostgreSQL, MySQL, SQLite (extendable to others)
* **Protocol:** MCP (Model Context Protocol) — Port **7766**

---

### 🔁 Core Flow

1. **Chat Input** → Interpret user prompt.
2. **Connect to Database (Tool)** → Establish connection(s).
3. **Get Schema (Tool)** → Retrieve and cache schema.
4. **Analyze Schema** → Understand relationships and metadata.
5. **Display Tables** → List available tables and columns.
6. **Generate Query (Tool)** → Build valid SQL via OpenAI + LangChain.
7. **Debug Mode** → Display SQL for confirmation.
8. **Execute Query (Tool)** → Safely execute SQL.
9. **Format Output (Tool)** → Render as formatted CLI table.
10. **Analyze Result (Tool)** → Summarize and explain findings.
11. **Visualize (Tool)** → Plot data into charts or graphs (CLI or Web UI).
12. **Multi-DB Support** → Query across multiple databases if connected.
13. **Web UI** → Real-time web dashboard for chat, query, and visualization.

---

### 🧩 Tools to Implement

| Tool Name             | Purpose                                              |
| --------------------- | ---------------------------------------------------- |
| `connect_db_tool`     | Connect to a database (Postgres/MySQL/SQLite)        |
| `get_schema_tool`     | Retrieve schema metadata                             |
| `analyze_schema_tool` | Analyze table structure and relationships            |
| `generate_query_tool` | Generate SQL query from natural language             |
| `execute_query_tool`  | Execute SQL safely and return results                |
| `format_table_tool`   | Display result as formatted CLI table                |
| `analyze_data_tool`   | Summarize and interpret results                      |
| `visualize_data_tool` | Generate graphs or charts from result sets           |
| `multi_db_query_tool` | Combine results from multiple DBs                    |
| `web_ui_tool`         | Serve real-time Web UI via FastAPI or MCP Web Socket |

---

### 🧠 Extended Features

#### 📊 **1. Data Visualization**

* Use **Plotly**, **Matplotlib**, or **Altair**.
* Visuals supported: Bar, Line, Scatter, Pie, Heatmap.
* Output formats:

  * CLI: ASCII-art preview (using Rich)
  * Web: Interactive HTML (rendered by FastAPI)
* Tool: `visualize_data_tool`
* Example usage:

  ```
  > show sales trend by month
  📈 Generating chart...
  🧠 Displaying bar chart (sales vs month)
  ```

#### 🗃️ **2. Multi-Database Queries**

* Allow connecting to multiple databases simultaneously.
* Support cross-database queries (merge/join on similar schema).
* Example:

  ```
  > compare sales between db_a and db_b
  ```
* Tool: `multi_db_query_tool`
* Behavior:

  * Retrieve data from both DBs.
  * Align schema where possible.
  * Combine results and analyze.

#### 🌐 **3. Web UI**

* Built using **FastAPI (Python)** backend + **React/Svelte** frontend.
* Features:

  * Chat interface for querying.
  * Schema explorer.
  * Real-time visualization dashboard.
* WebSocket or MCP streaming for live updates.
* Tool: `web_ui_tool`
* Route examples:

  * `/` → Chat interface
  * `/schema` → Database schema view
  * `/charts` → Visualization dashboard
  * `/api/query` → Execute query
  * `/api/analyze` → Generate insights

---

### ⚙️ MCP Integration

* Expose MCP endpoint on **port 7766**.
* Supported actions:

  * `connect_db`
  * `get_schema`
  * `generate_query`
  * `execute_query`
  * `visualize`
  * `analyze_results`
  * `multi_db_query`
* Context sharing:

  * Current database(s)
  * Schema state
  * Query history
  * Visualization metadata

---

### 🧠 Assistant Behavior Rules (for Cursor AI)

* Maintain **modular**, **typed**, and **documented** Python structure.
* For CLI: prefer **Typer**.
* For tables and visualization: use **Rich** and **Plotly**.
* For API: use **FastAPI** with async endpoints.
* For web: use **React or Svelte + Tailwind**.
* Always show SQL before execution if debug is enabled.
* Never run destructive SQL commands automatically.
* Include `--visualize` flag for visualization in CLI.
* Include `--web` flag to launch the Web UI mode.
* Write testable, small, independent functions.

---

### 📂 Recommended Folder Structure

```
myquery/
 ├─ cli/
 │   ├─ main.py
 │   └─ commands/
 │       ├─ chat.py
 │       ├─ connect.py
 │       ├─ query.py
 │       └─ visualize.py
 ├─ core/
 │   ├─ agent.py
 │   ├─ schema_analyzer.py
 │   ├─ query_generator.py
 │   ├─ data_analyzer.py
 │   ├─ visualizer.py
 │   ├─ multi_db_manager.py
 ├─ tools/
 │   ├─ connect_db_tool.py
 │   ├─ get_schema_tool.py
 │   ├─ generate_query_tool.py
 │   ├─ execute_query_tool.py
 │   ├─ format_table_tool.py
 │   ├─ analyze_data_tool.py
 │   ├─ visualize_data_tool.py
 │   ├─ multi_db_query_tool.py
 ├─ web/
 │   ├─ main.py (FastAPI)
 │   ├─ routes/
 │   │   ├─ api_query.py
 │   │   ├─ api_analyze.py
 │   │   └─ api_visualize.py
 │   ├─ static/
 │   ├─ templates/
 ├─ mcp/
 │   ├─ server.py
 │   ├─ protocol.py
 ├─ config/
 │   ├─ settings.py
 │   ├─ logging.py
 ├─ tests/
 │   ├─ test_tools.py
 │   ├─ test_agent.py
 │   ├─ test_web.py
 ├─ pyproject.toml
 ├─ README.md
```

---

### 🧩 Cursor Codegen Markers

Use these markers to guide Cursor when generating new code:

```python
# @cursor: define-tool connect_db_tool
# @cursor: define-tool get_schema_tool
# @cursor: define-tool generate_query_tool
# @cursor: define-tool execute_query_tool
# @cursor: define-tool format_table_tool
# @cursor: define-tool analyze_data_tool
# @cursor: define-tool visualize_data_tool
# @cursor: define-tool multi_db_query_tool
# @cursor: define-tool web_ui_tool
```

---

### ✅ Goals

* Enable **multi-modal database interaction**: text, query, and visualization.
* Provide **CLI + Web dual experience**.
* Maintain **clear, modular architecture**.
* Support **MCP-based orchestration** and **LangChain agentization**.
* Empower Cursor AI to autogenerate well-structured, type-safe Python code.

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
