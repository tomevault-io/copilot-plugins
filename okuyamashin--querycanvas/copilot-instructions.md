## querycanvas

> QueryCanvas is a database client extension for VS Code/Cursor with AI-powered features.

# Cursor Rules for QueryCanvas - Database Client

## Project Overview
QueryCanvas is a database client extension for VS Code/Cursor with AI-powered features.

---

## ⚠️ MOST IMPORTANT: SQL Display Options Syntax

When generating SQL with display options, follow these rules EXACTLY:

### @column directive (cell styling)
```sql
@column <name> type=int if<0:color=red
```
- Use `if<operator><value>:style` for conditions
- Example: `@column 売上 type=int if<0:color=red if>1000000:color=green`

### @row directive (entire row styling)
```sql
@row <column_name><operator><value>:<styles>
```
- **NO `if` keyword!**
- Use `==` (double equals) for equality
- Quote strings: `"value"` or `'value'`
- Use `bg` NOT `background`
- Example: `@row 曜日=="土":bg=#eeeeff`
- Example: `@row 売上>1000000:bg=#ccffcc,bold=true`

### @chart directive (graph visualization) 🆕
```sql
@chart type=line x=日付 y=売上,利益
@chart type=mixed x=月 y=売上:bar,目標:line
@chart type=pie x=店舗名 y=件数 colors="#FF6384,#36A2EB,#FFCE56,#4BC0C0,#9966FF"
```
- **Required:** `type`, `x` (or `xAxis`), `y` (or `yAxis`)
- **Chart types:** `line`, `bar`, `pie`, `area`, `scatter`, `mixed`
- **Y-axis:** Comma-separated for multiple series (e.g., `y=店舗A,店舗B,店舗C`)
- **Mixed charts:** Specify type for each series: `y=売上:bar,目標:line,達成率:line`
- **Optional:** `title="タイトル"`, `legend=true`, `grid=true`, `stacked=true`, `curve=smooth`
- **Colors (for pie charts):** `colors="#color1,#color2,#color3"` - Specify colors for each segment
- Example: `@chart type=line x=日付 y=小村井店,京成小岩店 title="店舗別売上推移"`
- Example: `@chart type=mixed x=月 y=売上:bar,目標:line title="売上と目標"`
- Example: `@chart type=pie x=店舗名 y=件数 colors="#FF6384,#36A2EB,#FFCE56" title="店舗別シェア"`

### ❌ WRONG Examples
```sql
@row if 曜日=土:background=#eee     ❌ NO! Has 'if', uses '=', uses 'background'
@row 国名=フランス:bg=#fee           ❌ NO! No quotes, uses single '='
@chart x=日付                       ❌ NO! Missing required 'type' and 'y'
```

### ✅ CORRECT Examples
```sql
@row 曜日=="土":bg=#eee              ✅ YES!
@row 国名=="フランス":bg=#fee        ✅ YES!
@row 売上>1000000:bg=#ccffcc         ✅ YES!
@chart type=line x=日付 y=売上       ✅ YES!
```

---

## 🎨 SQL Display Options - Quick Reference

### Two Types of Styling

#### 1. Column Styling (`@column`) - Styles individual cells
```sql
/**
 * @column 売上 type=int align=right format=number comma=true if<0:color=red
 */
```
- Uses `if<value:style` syntax for conditional styling
- Applies to individual cells in that column

#### 2. Row Styling (`@row`) - Styles entire rows
```sql
/**
 * @row 曜日=="土":bg=#eeeeff
 * @row 売上>1000000:bg=#ccffcc,bold=true
 */
```
- **CRITICAL:** Do NOT use `if` keyword
- Use `==` (double equals) for equality, NOT `=` (single)
- Strings MUST be quoted: `"値"` or `'値'`
- Use `bg` or `backgroundColor`, NOT `background`
- Applies styling to the entire row

### Common Mistakes ⚠️

| ❌ WRONG | ✅ CORRECT |
|----------|-----------|
| `@row if 曜日=土:background=#eee` | `@row 曜日=="土":bg=#eee` |
| `@row 国名=フランス:bg=#fee` | `@row 国名=="フランス":bg=#fee` |
| `@row 売上>1000:background=#cfc` | `@row 売上>1000:bg=#cfc` |

### Syntax Comparison

| Feature | Column Style | Row Style |
|---------|-------------|-----------|
| Directive | `@column` | `@row` |
| Conditional | `if<0:color=red` | `売上<0:bg=#fcc` |
| `if` keyword | ✅ YES (use `if`) | ❌ NO (`if` not used) |
| Equality | `==` or single `=` | `==` (double only) |
| String values | No quotes needed in options | MUST quote: `"value"` |
| Example | `@column 損益 type=int if<0:color=red` | `@row 曜日=="土":bg=#eef` |

---
## 🤖 Cursor AI Integration

This extension is designed to work seamlessly with Cursor AI. You can edit SQL queries by modifying the session file.

### SQL Session File

**Location:** `.vscode/querycanvas-session.json`

**What it contains:**
- Current SQL query in the editor
- Active database connection ID
- Last update timestamp

**How Cursor AI can help:**

1. **Edit SQL directly in session file:**
   - User can ask: "Modify the SQL in the session to add a WHERE clause"
   - Cursor AI edits `.vscode/querycanvas-session.json`
   - Changes reflect in Database Client UI immediately (via file watcher)

2. **Generate optimized SQL:**
   - User can ask: "Rewrite this SQL with display options"
   - Cursor AI updates the `sqlInput` field with formatted SQL

3. **Add display options:**
   - User can ask: "Add display formatting to this query"
   - Cursor AI inserts `/** @column ... */` comments

**Example session file:**
```json
{
  "connectionId": "production-db",
  "isConnected": false,
  "sqlInput": "/**\n * @column amount align=right format=number comma=true\n */\nSELECT amount FROM orders",
  "lastUpdated": "2025-12-28T12:00:00.000Z"
}
```

**Workflow:**
```
User: "Add display options to format the 'price' column"
  ↓
Cursor AI: Edits .vscode/querycanvas-session.json
  ↓
File watcher: Detects change
  ↓
UI: Updates automatically
  ↓
User: Sees formatted SQL in Database Client
```

## SQL Display Options Feature

This extension supports custom display formatting through SQL comments.

### Syntax
Use `/** ... */` comments at the beginning or end of SQL queries with `@column` and `@row` directives:

```sql
/**
 * @column <column_name> <option>=<value> <option>=<value> ...
 * @row <column_name><operator><value>:<styles>
 */
SELECT ...
```

### Supported Options

#### Text Alignment
- `align=left|center|right` - Text alignment in table cells

#### Number Formatting
- `format=number` - Format as number
- `comma=true|false` - Add thousands separator (1,234,567)
- `decimal=N` - Number of decimal places (e.g., decimal=2 → 123.45)

#### Datetime Formatting
- `format=datetime` - Format as datetime
- `pattern=<format>` - Date format pattern
  - `yyyy` = 4-digit year
  - `MM` = 2-digit month
  - `dd` = 2-digit day
  - `HH` = 2-digit hour (24h)
  - `mm` = 2-digit minute
  - `ss` = 2-digit second
  - Example: `pattern=yyyy/MM/dd_HH:mm:ss`
  - **Note:** Use double quotes if pattern contains spaces: `pattern="yyyy-MM-dd HH:mm:ss"`

#### Column Styling
- `width=<size>` - Column width (e.g., `width=200px`)
- `color=<color>` - Text color (e.g., `color=#ff0000`)
- `bg=<color>` or `backgroundColor=<color>` - Background color
- `bold=true` - Make text bold

#### Data Type & Conditional Styling 🆕
- `type=int|float|decimal|text` - Specify column data type
- `if<演算子><値>:<スタイル>` - Conditional styling based on cell value (within @column directive)
  - Operators: `<`, `>`, `<=`, `>=`, `==`, `!=`
  - Styles: `color=<color>`, `bg=<color>`, `bold=true`, `fontWeight=<value>`
  - Example: `if<0:color=red` (negative values in red)
  - Example: `if>1000:bold=true` (values over 1000 in bold)
  - Example: `if<=0:color=#999999 if>=10000:color=#ff0000,bold=true` (multiple conditions)
  - **Note:** This `if` syntax is ONLY for @column directives, NOT for @row directives

#### Row Styling 🆕
- `@row <column_name><operator><value>:<styles>` - Style entire rows based on column values
  - **IMPORTANT:** Do NOT use `if` keyword. Use direct comparison: `@row 列名==値` (NOT `@row if 列名=値`)
  - Operators: `<`, `>`, `<=`, `>=`, `==`, `!=` (use `==` for equality, NOT single `=`)
  - Values: Numbers (e.g., `1000`) or strings with quotes (e.g., `"フランス"`, `'completed'`)
  - Styles: `color=<color>`, `bg=<color>` or `backgroundColor=<color>`, `bold=true`, `fontWeight=<value>`
  - Example: `@row 国名=="フランス":color=#ff0000,bg=#ffeeee` (rows with country="フランス" in red)
  - Example: `@row 売上>1000000:bg=#ccffcc,bold=true` (rows with sales>1M in green)
  - Example: `@row ステータス=="完了":bg=#d4edda,color=#155724` (completed rows in green)
  - **Common mistakes to avoid:**
    - ❌ `@row if 曜日=土:background=#eeeeff` (WRONG: has `if` keyword, uses `=` instead of `==`, uses `background` instead of `bg`)
    - ✅ `@row 曜日=="土":bg=#eeeeff` (CORRECT: no `if`, uses `==`, uses `bg`)

### Examples

#### Financial Report
```sql
/**
 * @column 店舗名 width=150px
 * @column 売上 align=right format=number comma=true
 * @column 前年比 align=right format=number decimal=1
 * @column 更新日時 format=datetime pattern=yyyy/MM/dd_HH:mm
 */
SELECT 店舗名, 売上, 前年比, 更新日時 FROM sales_report;
```

#### Styled Table
```sql
/**
 * @column ID align=right
 * @column ステータス color=#00ff00 bold=true
 * @column 金額 align=right format=number comma=true decimal=2
 * @column 作成日時 format=datetime pattern=yyyy/MM/dd
 */
SELECT ID, ステータス, 金額, 作成日時 FROM orders;
```

#### Conditional Styling Example 🆕
```sql
/**
 * @column 売上 type=int align=right format=number comma=true if<0:color=red,bold=true if>1000000:color=blue,bold=true
 * @column 在庫数 type=int align=right if<=0:color=red,bold=true if<=10:color=orange if>100:color=green
 * @column 達成率 type=float align=right format=number decimal=1 if<80:color=red if>=100:color=green,bold=true
 */
SELECT 売上, 在庫数, 達成率 FROM performance_data;
```

#### Row Styling Example 🆕
```sql
/**
 * @row 国名=="フランス":color=#ff0000,bg=#ffeeee
 * @row 国名=="日本":color=#0000ff,bg=#eeeeff
 * @row 売上>1000000:bg=#ccffcc,bold=true
 * @column 売上 align=right format=number comma=true
 */
SELECT 国名, 都市, 売上, 担当者 FROM sales_data;
```

#### Row + Column Styling Combined 🆕
```sql
/**
 * @row 達成率>=100:bg=#e8f5e9
 * @row 達成率<80:bg=#ffebee
 * @column 売上 type=int align=right format=number comma=true if<0:color=red
 * @column 達成率 type=float align=right format=number decimal=1 if<80:color=red if>=100:color=green,bold=true
 */
SELECT 店舗名, 売上, 達成率, 前年比 FROM performance_dashboard;
```

### Implementation Details
- Parser: `src/sqlCommentParser.ts`
- Options are extracted in `DatabaseClientPanel._handleExecuteQuery()`
- Formatting applied in Webview's `handleQueryResult()` function
- Number formatting uses `formatValue()` helper
- Datetime formatting uses `formatDateTime()` helper
- Column styling uses `generateColumnStyle()` helper (for headers)
- Conditional styling uses `generateConditionalStyle()` helper (for cells, evaluates conditions based on cell values)
- Row styling uses `generateRowStyle()` helper (for rows, evaluates conditions based on row data) 🆕

## Coding Guidelines

### TypeScript
- Use strict typing
- Prefer interfaces over types for extensibility
- Document public APIs with JSDoc comments

### SQL Validation
- Only read-only queries allowed (SELECT, SHOW, DESC, EXPLAIN)
- INSERT, UPDATE, DELETE are blocked by `SqlValidator`

### Session Persistence
- SQL input auto-saved to `.vscode/db-client-session.json`
- File watcher syncs external changes (Cursor edits)
- Query results auto-saved to `query-results/` directory

### Internationalization
- English is default language
- Japanese is supported
- Translation files: `src/i18n/en.json`, `src/i18n/ja.json`
- Schema documents adapt to VS Code language setting

## File Structure
- `src/databaseClientPanel.ts` - Main Webview panel
- `src/database/` - Database connection layer
- `src/i18n/` - Translation files
- `src/sqlCommentParser.ts` - Display options parser
- `src/sqlFormatter.ts` - SQL beautifier
- `src/sqlValidator.ts` - SQL security validator

## 📋 Clipboard Copy to PowerPoint/Excel/Word Feature 🆕

After executing a query, users can copy results to clipboard in two formats:

### TSV Copy (Tab-Separated Values)
- **Button:** "📋 TSVコピー"
- **Format:** Plain text with tab separators
- **Use case:** Simple data transfer to any application
- **Preserves:** Data only (no styling)
- **Compatible with:** PowerPoint, Excel, Word, Google Sheets, etc.

### HTML Copy (Rich Format)
- **Button:** "📋 HTMLコピー"
- **Format:** HTML table with inline styles
- **Use case:** Rich presentations and reports
- **Preserves:** 
  - Colors, bold, and other text styles
  - Conditional styling (red negatives, green positives, etc.)
  - Number formatting (commas, decimal places)
  - Alignment and column widths
- **Compatible with:** PowerPoint, Excel, Word, Outlook

### When to Recommend

**Recommend HTML Copy when:**
- User needs styled tables for presentations
- Query uses conditional styling (`if<0:color=red`)
- Query uses number/datetime formatting
- User mentions PowerPoint, Excel, or creating reports

**Recommend TSV Copy when:**
- User needs simple data transfer
- Compatibility is a concern
- No styling is needed

### Example Workflow for Presentations

```sql
/**
 * @column 売上 type=int align=right format=number comma=true if<0:color=red if>2000000:color=green,bold=true
 * @column 達成率 type=float align=right format=number decimal=1 if<80:color=red if>=100:color=green,bold=true
 * @column 更新日時 format=datetime pattern=yyyy/MM/dd
 */
SELECT 店舗名, 売上, 達成率, 更新日時
FROM sales_dashboard
ORDER BY 売上 DESC
LIMIT 10;
```

→ Execute → Click "📋 HTMLコピー" → Paste in PowerPoint → Beautiful styled table!

### User Prompts You Might See

- "PowerPointに貼り付けられる形式でコピーしたい"
  → Suggest HTML Copy for styled output
  
- "このクエリ結果をExcelで使いたい"
  → Suggest TSV Copy for data, HTML Copy for styled reports
  
- "プレゼン資料を作りたい"
  → Suggest adding display options + conditional styling + HTML Copy
  
- "資料用にデータをコピーしたい"
  → Ask if styling is needed, then recommend HTML or TSV Copy

### Implementation Details
- TSV copy: `copyTableAsTSV()` function in `databaseClientPanel.ts`
- HTML copy: `copyTableAsHTML()` function in `databaseClientPanel.ts`
- Buttons appear automatically after query execution
- Uses Clipboard API: `navigator.clipboard.write()` for HTML, `writeText()` for TSV

### Documentation
- User guide: `docs/POWERPOINT-COPY-GUIDE.md`
- Testing guide: `docs/TESTING-POWERPOINT-COPY.md`

## When Helping with SQL Queries

1. Always suggest using display options for better presentation
2. Use `align=right` for numeric columns
3. Use `format=number comma=true` for currency/large numbers
4. Use `format=datetime pattern=...` for timestamps
5. Use conditional styling for highlighting important values (e.g., `if<0:color=red` for negative numbers)
6. Specify `type=int|float|decimal` when using conditional styling
7. **🆕 Use row styling (`@row`) to highlight entire rows based on conditions**
8. **🆕 Combine row and column styling for sophisticated visual presentations**
9. **🆕 Use graph visualization (`@chart`) for time-series data and trend analysis**
10. Keep column names in original language (often Japanese)
11. Remember this is a read-only client (no INSERT/UPDATE/DELETE)
12. **Mention clipboard copy feature when user needs to create reports or presentations**
13. **Suggest HTML Copy to preserve styling in PowerPoint/Excel**
14. **Suggest chart visualization for data with temporal or categorical patterns**

## ⚠️ CRITICAL: Row Styling Syntax

**When user asks about row styling or @row directives, ALWAYS use this syntax:**

```sql
@row <column_name><operator><value>:<styles>
```

**Rules:**
- ❌ Do NOT use `if` keyword
- ✅ Use `==` (double equals) for equality
- ✅ Quote string values: `"土"`, `'完了'`
- ✅ Use `bg` or `backgroundColor` (NOT `background`)

**Examples:**
```sql
@row 曜日=="土":bg=#eeeeff
@row 売上>1000000:bg=#ccffcc,bold=true
@row ステータス=="完了":bg=#d4edda,color=#155724
```

**NOT:**
```sql
@row if 曜日=土:background=#eeeeff  ❌ WRONG!
```

## 📈 Graph Visualization Feature 🆕

QueryCanvas now supports interactive chart visualization using Chart.js.

### When to Suggest Chart Visualization

**Recommend charts when:**
- User has time-series data (dates, timestamps)
- User wants to compare multiple categories or stores
- Data shows trends or patterns
- User mentions: "グラフ", "チャート", "可視化", "トレンド", "推移"

### Chart Directive Syntax

```sql
/**
 * @chart type=<type> x=<x_column> y=<y_columns> [options]
 */
```

**Required:**
- `type`: Chart type (`line`, `bar`, `pie`, `area`, `scatter`)
- `x` or `xAxis`: Column name for X-axis
- `y` or `yAxis`: Column name(s) for Y-axis (comma-separated for multiple series)

**Optional:**
- `title="タイトル"`: Chart title
- `legend=true|false`: Show/hide legend (default: true)
- `grid=true|false`: Show/hide grid lines (default: true)
- `stacked=true|false`: Stack bars (for bar charts only)
- `curve=smooth|straight`: Line curve style (for line charts, default: smooth)

### Chart Types and Use Cases

| Type | Use Case | Example |
|------|----------|---------|
| `line` | Time-series, trends | Daily sales, monthly revenue |
| `bar` | Category comparison | Store ranking, product sales |
| `area` | Cumulative data | Stacked revenue streams |
| `pie` | Proportions | Market share, category breakdown |
| `scatter` | Correlation | Price vs. quantity |
| `mixed` 🆕 | Combined visualization | Sales (bar) + Target (line) |

**Note for pie charts:** Use `colors` parameter to specify colors for each segment:
```sql
@chart type=pie x=店舗名 y=件数 colors="#FF6384,#36A2EB,#FFCE56,#4BC0C0,#9966FF"
```

### Mixed Chart Examples 🆕

Combine different chart types in one visualization:

```sql
/**
 * @chart type=mixed x=月 y=売上:bar,目標:line title="売上実績と目標"
 * @column 売上 color="#36A2EB"
 * @column 目標 color="#FF6384"
 */
SELECT 
  DATE_FORMAT(order_date, '%Y-%m') AS '月',
  SUM(amount) AS '売上',
  10000000 AS '目標'
FROM sales
GROUP BY DATE_FORMAT(order_date, '%Y-%m');
```

**Common mixed chart patterns:**
- `y=売上:bar,目標:line` - Actual (bar) vs Target (line)
- `y=注文数:bar,平均単価:line` - Volume (bar) + Average (line)
- `y=今月:bar,先月:bar,トレンド:line` - Period comparison + trend

### Color Integration

Colors specified in `@column` directives are automatically applied to chart series:

```sql
/**
 * @chart type=line x=日付 y=店舗A,店舗B
 * @column 店舗A color="#FF6384"
 * @column 店舗B color="#36A2EB"
 */
```

### Complete Example

```sql
/**
 * @chart type=line x=日付 y=小村井店,京成小岩店 title="店舗別売上推移"
 * @row 曜日=="土":bg=#eeeeff
 * @row 曜日=="日":bg=#ffeeee
 * @column 小村井店 type=int align=right format=number comma=true color="#FF0000"
 * @column 京成小岩店 type=int align=right format=number comma=true color="#008800"
 */
SELECT 
  DATE_FORMAT(YMD_CREATED, '%Y/%m/%d') AS '日付',
  CASE DAYOFWEEK(YMD_CREATED)
    WHEN 1 THEN '日' WHEN 2 THEN '月' WHEN 3 THEN '火'
    WHEN 4 THEN '水' WHEN 5 THEN '木' WHEN 6 THEN '金' WHEN 7 THEN '土'
  END AS '曜日',
  SUM(CASE WHEN SHOP_NAME = '小村井店' THEN 1 ELSE 0 END) AS '小村井店',
  SUM(CASE WHEN SHOP_NAME = '京成小岩店' THEN 1 ELSE 0 END) AS '京成小岩店'
FROM sales_data
WHERE YMD_CREATED LIKE '202508%'
GROUP BY YMD_CREATED
ORDER BY YMD_CREATED;
```

### Best Practices for Chart Visualization

1. **Use meaningful column names** in Japanese for better readability
2. **Specify colors** with `@column` to match corporate/brand colors
3. **Add chart title** for context: `title="店舗別売上推移"`
4. **Combine with row styling** to highlight important data points in table view
5. **Format numbers** with `format=number comma=true` for professional appearance
6. **Choose appropriate chart type**:
   - Line charts for trends over time
   - Bar charts for comparing categories
   - Pie charts for showing proportions (use LIMIT 10 for clarity)
7. **Use multiple Y-axis series** to compare related metrics

### Common User Requests and Responses

**User:** "売上のトレンドをグラフで見たい"
**AI Response:** Suggest `@chart type=line` with date as X-axis and sales as Y-axis

**User:** "店舗別の比較をしたい"
**AI Response:** Suggest `@chart type=bar` or multiple series line chart

**User:** "プレゼン資料を作りたい"
**AI Response:** 
1. Suggest chart with professional styling
2. Add row styling for emphasis
3. Mention **📊 グラフコピー** button for PowerPoint

**User:** "このデータを可視化したい"
**AI Response:**
1. Analyze data structure (time-series? categories?)
2. Suggest appropriate chart type
3. Provide complete SQL with `@chart` and styling

### UI Behavior

When `@chart` is specified:
- **📊 テーブル** and **📈 グラフ** toggle buttons appear
- Users can switch between table view and chart view
- Both views show the same data in different formats

### Copy Chart to PowerPoint/Word 🆕

After displaying a chart, users can copy it as an image:

**Steps:**
1. Execute SQL with `@chart` directive
2. Click **📈 グラフ** to view chart
3. Click **📊 グラフコピー** button
4. Open PowerPoint/Word/Keynote
5. Paste (`Cmd+V` / `Ctrl+V`)

**Features:**
- High-quality PNG image
- Includes title, legend, colors, and all styling
- White background (not transparent)
- Works in PowerPoint, Word, Keynote, Google Slides

**When to Recommend:**
- User mentions "プレゼン", "PowerPoint", "資料作成"
- User needs charts for reports or presentations
- User wants to share visual analysis results

**Example Workflow:**
```sql
/**
 * @chart type=line x=日付 y=小村井店,京成小岩店 title="店舗別売上推移"
 * @column 小村井店 color="#FF0000"
 * @column 京成小岩店 color="#008800"
 */
SELECT ...
```
→ Execute → Click **📈 グラフ** → Click **📊 グラフコピー** → Paste in PowerPoint → Done!

### Documentation

- Full guide: `docs/CHART-VISUALIZATION-GUIDE.md`
- Technical specs: Chart.js v4.4.1 via CDN
- Examples: `docs/examples/chart-examples.sql` (10 samples)

---
> Source: [okuyamashin/querycanvas](https://github.com/okuyamashin/querycanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
