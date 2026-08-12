## verilog-rule

> Portable Verilog-2001 RTL rules (FSM, latch avoidance, unified if/else-if for seq+comb always, parallel-if split, cross-always multi-driven, ROM/SRAM read contracts). Copy to any project; chip-specific golden paths belong in separate rules.


# Verilog RTL 設計規範

**適用範圍**：可複製到任意 Verilog-2001 RTL 專案。全文以**可合成、可維護**為主；**§7 起**含常見 **TSMC 1P SRAM／ROM `CLK(~clk)`** 範例與除錯備忘——若你的 macro／供應商不同，保留**契約三行**與**ADDR/USE 兩段**思想，替換接腳與 latency 即可。晶片或專案專用的 golden 路徑、檔名對照請放在**另一份**規則，勿寫進本檔。

## 語言規則
- 僅使用 Verilog-2001 語法，禁止 SystemVerilog 特性（logic, always_ff, always_comb, interface, typedef, enum, struct, package, assertions）
- 所有 RTL 必須可合成（synthesizable），禁止 initial（硬體邏輯）、# delay、fork/join、force/release
- **`reg` 不可在 `always` 區塊內宣告**（含 `if` 內）；暫存器一律在 module 層級宣告

## 程式結構
- Parameter 宣告在前，reg 宣告居中，always block 在後
- 不交錯 reg 宣告與 always block（`assign`／`always` 之後不得再新增 `reg`；debug 用暫存器須與其他 `reg` 同區）
- 每個 always block 前必須有一行用途註解
- **時序與組合** `always` 一律遵守 **「`if` / `else if` 鏈（共通規範）」**；若同層曾有平行多個 `if`，依該節拆分或改寫，拆分後須排除 **跨 `always` 多驅動**；組合另須 **區塊頂端 default**，避免 latch

## 模組化設計
- 採用 top-down 架構：top → controller → datapath modules
- 介面必須定義 clk, reset, start, busy, done, data_in, data_out
- 大邏輯拆分為 controller / compute core / pipeline stage / memory interface

## FSM 設計
- FSM 必須獨立為三段式：state register / next-state logic / output logic
- 使用 parameter 定義狀態名稱
- **`state` 暫存** 與 **`next_state` 組合** 各一個 `always`；**datapath／控制輸出** 可拆成多個 `posedge` `always`（见下方 **「`if` / `else if` 鏈」** 與拆分節），但 **不可** 與 `state`／`next_state` 寫在同一 block

## 時序與組合邏輯
- `always @(posedge clk)` 用於 sequential（**時序 block**），`always @(*)` 用於 combinational（**組合 block**）
- 優先使用同步 reset
- Sequential 用 `<=`（non-blocking），combinational 用 `=`（blocking）
- 組合邏輯必須覆蓋所有輸出，**禁止 latch 推論**（細則见下節）
- **時序與組合 block 皆須**遵守下方 **「`if` / `else if` 鏈（共通規範）」**——不是只約束其中一種

### `if` / `else if` 鏈（時序與組合 always 共通，必守）

**適用範圍**：凡 `always @(posedge clk)`（時序）與 `always @(*)`（組合）內的條件分支，**同一套結構規範**；不可只對組合要求互斥、時序卻放任同層平行 `if`。

#### 同層結構（定義）

1. **同一巢狀層級**內，對該層的互斥／優先決策只能有一個開頭的 `if`；其後同層條件**必須**寫成 `else if`（需要時以 `else` 收尾）。
2. **巢狀**時：外層是一條 `if`／`else if` 鏈；進入某一分支後，**內層再各自**遵守「一個 `if` + 其後皆 `else if`」。不可在同一層平行寫：
   ```verilog
   // BAD (same nesting level): two sibling ifs
   if (cond_a) ...;
   if (cond_b) ...;   // prefer: else if (cond_b); split always ONLY if LHS sets differ
   ```
3. **`case`／`casex`** 與 `if` 鏈並存時：同一組輸出在「`case` 分支」與「區塊上方的 `if`」之間若會同拍／同拍組合覆寫，須合併為**單一優先鏈**或依拆分節移到不同 `always`（時序）／先 default 再單一覆寫路徑（組合）。

#### 為何時序也要遵守（勿誤讀成「時序可平行 if」）

| | 組合 `always @(*)` | 時序 `always @(posedge clk)` |
|--|-------------------|------------------------------|
| 同層平行多個 `if` | **禁止**（latch、多驅動、路徑不明） | **禁止**（同 reg 多段 NBA 時順序即語意；與 `case` 混寫易 GLS／X） |
| 合規寫法（預設） | 頂端 default + 單一 `if`／`else if` 鏈，或 `case`+`default` | **同一** `always` 內改成 `if`／`else if` 優先鏈 |
| 合規寫法（例外） | — | **僅當**各分支 LHS **互不相同**且須同拍並行更新 → **拆成多個** `always`（每 block 各自一條鏈） |

> 舊表述「時序平行 `if` 無 `else`＝enable FF 故允許」**僅**指「**單一** `if (en)` 更新、條件不成立則 hold」這種**一個** enable；**不表示**允許同層再並排第二個獨立 `if`。遇到同層 sibling `if`：**先**改成同一 block 的 `if`／`else if`；**只有** LHS 集合不同、又必須同拍都更新時，才拆 `always`（勿把本可互斥的條件硬拆，也勿對須並行的獨立 LHS 誤用 `else if` 串成互斥）。

#### 允許的「單一 enable」（時序專用，不是平行決策）

```verilog
// OK: one enable; hold when en==0  →  enable FF (not a comb latch)
always @(posedge clk) begin
    if (reset)
        foo <= 'd0;
    else if (en)
        foo <= next_foo;
end
```

```verilog
// OK: one-cycle pulse — DFF samples fire each cycle (fire already 1-cycle)
always @(posedge clk) begin
    if (reset)
        pulse <= 1'b0;
    else
        pulse <= fire;
end
```

> `fire` 須為組合／已對齊的單週期條件。**避免**「`else` 裡先 `pulse<=0` 再 `if (fire) pulse<=1`」靠同拍後寫覆蓋；若條件不只 `fire` 一項，再用 `if`／`else if`／`else` 展開（见下方脈衝模板）。
```verilog
// BAD: same level, sibling ifs (seq or comb) — do not leave like this
always @(posedge clk) begin
    if (reset) begin /* ... */ end
    else begin
        if (cond_a) /* drive set A */ ;
        if (cond_b) /* drive set B */ ;
        if (cond_c) /* drive set C */ ;
    end
end
```

對上例 **BAD** 的改法（順序固定）：

1. **預設**：A／B／C 改成**同一個** `always` 內的 `if`／`else if`／`else` 優先鏈（條件互斥或需排優先時皆然；LHS 有重疊時**必須**如此，**禁止**拆成多個 `always` 以免多驅動）。
2. **例外才拆**：僅當 A／B／C 的 LHS **互不相同**，且語意上須**同拍並行**更新（例如管線各級）——此時**不可**用 `else if` 強行互斥（會少更新某一級），應拆成**多個獨立時序 `always`**，每個內部仍是各自的 `if`／`else if` 鏈。

```verilog
// OK (default): same always, if / else if priority chain
always @(posedge clk) begin
    if (reset)
        cnt <= 'd0;
    else if (unit_done)
        cnt <= 'd0;
    else if (stream_en)
        cnt <= cnt + 1'b1;
end
```

```verilog
// OK (exception): LHS disjoint + must update in parallel → one always per LHS set
// Purpose: pipeline stage A
always @(posedge clk) begin
    if (reset) /* clear A */ ;
    else if (mac_valid) /* drive set A only */ ;
end
// Purpose: pipeline stage B
always @(posedge clk) begin
    if (reset) /* clear B */ ;
    else if (compute_valid) /* drive set B only */ ;
end
```
#### 組合 block 額外義務

- 仍須遵守 **「避免產生 latch」**：頂端 default、`case` 有 `default`、路徑賦值齊全。
- Port mux／多路請求：同層必須 `if`／`else if` 互斥（见 §8.4）；**禁止**兩個 `if` 可能同時成立。

### 避免產生 latch（組合邏輯完整性，必守）

綜合工具在 **`always @(*)`**（或對敏感列表內所有變化都會執行的組合 `always`）中，若某個被賦值的 `reg`／`wire` **並非每一條執行路徑都有賦值**，會推論 **latch** 以「記住上一拍值」。**一律禁止** 依賴此類 latch；須改寫為明確暫存器（`posedge clk`）或補齊賦值。

#### 常見成因（寫 code 時逐項自查）

| 問題 | 說明 |
|------|------|
| **`if` 無 `else`** | 條件不成立時該訊號無賦值 → latch |
| **`case` 無 `default`** | 未列舉到的編碼無賦值 → latch |
| **巢狀分支不全** | 外層 `if` 成立、內層只覆蓋部分情況，其餘路徑未賦值 |
| **區塊開頭無預設值** | 多個 `if`／`case` 分支各自賦值，但漏掉某一組合條件 |
| **同一訊號多驅動** | 組合 `always` 與另一 `always`／`assign` 同時驅動（綜合／仿真行為混亂，亦可能伴隨不完整路徑） |
| **在 `always` 內宣告 `reg`** | 已禁止（见「語言規則」）；亦易造成路徑覆蓋難以檢查 |

#### 推薦寫法（組合 `always @(*)`）

1. **區塊最上方先對「本 block 驅動的所有輸出」寫預設賦值**（安全 idle 值），再寫 `if`／`case` 覆寫；預設須與 FSM／介面規格一致（例如 `next_state = state`、`busy = 0`、`mac_en = 0`）。
2. **同層只用一條 `if`／`else if`／`else` 鏈**（见共通規範）；每個決策分支路徑對輸出賦值齊全，或已靠頂端 default 覆蓋。避免「僅在條件成立時才賦值」且無 default。
3. **`case`／`casex` 必須有 `default`**；`case` 的每個分支須對**同一組輸出**給齊賦值，或已在區塊開頭 default 過。
4. **一個 `always` 只負責一類邏輯**（例如僅 next-state、僅 MAC 控制）；勿與無關 datapath 混寫，以免漏賦值。
5. **FSM 次態**：組合 block 內 `next_state` 先預設為 `state`（或 `IDLE`），再在 `case (state)` 覆寫；避免「某些 state 未寫 next_state」。
6. 能用 **`assign`** 表達的純組合式，優先 `assign`，減少 `always @(*)` 分支遺漏風險。

#### 時序 `always @(posedge clk)` 與 latch 的區別

- 在 **時序** block 中**單一** `if (en)` 內更新某 `reg`、無 `else` → 綜合為 **帶 enable 的 D 暫存器（hold）**，屬預期行為，**不是** 本节禁止的組合 latch；但仍須遵守共通規範：**同層不可再並排另一個 sibling `if`**。
- 若某值**必須在組合邏輯當拍就確定**（例如送進同拍 MAC 的 `mac_en`、`w_addr`），**不可** 放在「有時賦值、有時不賦值」的 `always @(*)`；應 default 賦齊，或改到時序 block 並 **對齊延遲一拍**（见「記憶體／ROM 讀取」）。

#### 修改／除錯時

- 綜合報告 **Inferred latch**、**latch inferred for …** 時，先定位對應 `always @(*)`，補 **default + 缺漏分支**，勿僅改最終輸出節點。
- 仿真若見組合輸出為 **X** 或「上一拍殘值」，先查該 block 是否在部分條件下**未賦值**（與 golden 對拍無關前也應先排除）。

### 面積/延遲
- 說明設計是 parallel / partially parallel / multi-cycle iterative
- 估算 MAC 數量、cycle 數、pipeline 深度

### 平行 `if` 的拆分與優先鏈（實作共通規範，必守）

本節是上一節 **「`if` / `else if` 鏈」** 的落地：發現同層平行 `if` 時如何改寫。**時序與組合皆適用**；組合改寫時另須保住 default／無 latch。

單一 `always`（尤其 `posedge clk`）內若有多個**平行** `if`（彼此**不是** `else if` 鏈），且常與 `case (state)` 或其它 `case` **寫在同一區塊**，常見後果：

- 違反共通的同層 `if`／`else if` 規範；
- 綜合把 **狀態、使能、計數器** 與 **暫存器 Q 的回授** 揉成大片組合邏輯；
- **Gate-level**（或含 async 控制腳的 FF）仿真出現 **X**、使能／清除腳異常、FSM「卡住」；
- 同一 `reg` 同拍多個 NBA 賦值時，**敘述順序**即語意；拆成多個 `always` 後若未還原優先順序，**功能會變**。

重構目標：**行為與重構前 RTL sim 一致**（有參考模型則一併對拍）；拆分本身不解決時序，但可降低上述風險並符合共通 `if`／`else if` 規範。

#### 與組合 latch／enable FF 的差別（勿混淆）

| | 組合 `always @(*)` | 時序 `always @(posedge clk)` |
|--|-------------------|------------------------------|
| 同層平行多個 `if` | **禁止**（latch／多驅動） | **禁止**（改 `else if` 或拆 `always`） |
| **單一** `if (en)` 無 `else` | 易 latch（禁止，除非已 default） | **允許**＝enable FF hold（見共通規範「單一 enable」） |
| 本節要處理的問題 | 漏賦值路徑 + 同層平行 `if` | **同層平行 `if`**、**多驅動**、**Q 回授**、**NBA 順序** |

#### 何時拆、何時不拆

| 情況 | 作法 |
|------|------|
| **同層 sibling `if`，條件互斥或需排優先**（LHS 可重疊或不重疊） | **預設**：留在**同一個** `always`，改成 **`if` / `else if` 鏈** 並註明優先順序 |
| **同一 `reg` 同拍可能被多段寫入**（LHS 重疊） | **禁止**拆成多個 `always`；**必須**單一 block + `if`／`else if` |
| **LHS 互不相同且須同拍並行**（例：管線 `mac_*`／`compute_*`／`acc_*`） | **例外才拆**到各自 `always`；每 block 前一行用途註解；**每個**內部仍是 `if`／`else if`（**禁止**同 block 平行 `if`；也**禁止**誤用 `else if` 把本該並行的更新串成互斥） |
| **單週期脈衝／strobe**（`start`、`done`、`valid`、`rd_en`） | 獨立 `always`：`reset` 後每拍先清預設值，再在條件成立時置位 |
| **重構前有多段敘述、同拍覆寫同一 reg** | 用 **`else if` 順序** 還原「誰覆寫誰」（见下節模式表）；勿拆到會多驅動的多個 `always` |
| **shadow／latch 暫存**（例：從 SRAM `Q` 鎖一筆供後續拍使用） | **獨立** `posedge` `always`；**僅此 block** 驅動該 `reg`（含 `reset`） |
| **大型 datapath `always` 的 `reset` 分支** | **勿** 對已在專用 `always` 驅動的 `reg` 再寫 `<= 0`（见下節「跨 always 多驅動」） |
| **組合 port mux／多路請求** | **同一** `always @(*)` 內用 `if`／`else if` 互斥；頂端 idle default（见 §8.4） |

#### 還原 NBA 優先順序（通用模式）

重構時對照**舊檔敘述順序**（仿真器對同一 `always` 內多個 `<=` 以區塊順序決定最終 NBA）。常見模式：

| 模式 | 舊版典型寫法 | 拆分後 `else if` 建議（`reset`／`ST_IDLE` 清零仍最前） |
|------|--------------|------------------------------------------------------|
| **A. `case` 覆寫前面的 `if`** | 區塊上方 `if (inc_en) ptr<=ptr+1;`，下方 `case` 內 `if (phase_done) ptr<=0` | **完成／重置條件先於遞增**：`else if (phase_done) …` **早於** `else if (inc_en) …` |
| **B. 進入子狀態時先清零指標** | 上方 `if (stream_en) … phase`，下方 `case` 內 `if (!unit_busy) ptr<=0` | **`!unit_busy`（或等同 idle）重置先於 stream** |
| **C. 同 state 內「完成」與「進度」並列** | `case` 內：`if (stream…) …` 與平行 `if (unit_done) cnt<=0` | **`unit_done` 先於 stream**：`if (unit_done) … else if (stream…) …` |
| **D. 外層區塊 + 內層 `case` 同拍改同一組 reg** | `if (state==ST_X) { 更新 A,B,C; }` 後接 `case(state)` 再改 A,B | **維持同一 `always` 且保留原語句先後**；或合併為單一 `if (state==ST_X)` 並依原順序排列 |

> 模式代號僅供對照；實作時在註解寫「對應舊版第 N 段／模式 X」，勿依賴專案專有名稱。

#### 推薦拆分粒度（controller／datapath）

1. **握手輸出**（`done`、`err`、`busy` 的 registered 版本）— 可獨立 `always`
2. **對外或子模組的單週期脈衝**（`*_start`、`*_kick`）— 一類一個 `always`（每拍 default 無效）
3. **計數器、指標、phase bit** — 依模式表拆，或單一 `always` + `else if` 鏈
4. **記憶體寫入 latch**（`wr_addr_lat`、`wr_data_lat`、`wr_pulse`）— 與 FSM `state` 分離
5. **同拍強耦合**（多個 reg 在同一拍互相依賴更新）— **留在同一** `always`
6. **僅隨 `case (state)` 變化的模式／索引暫存** — 可獨立 `always`

#### 脈衝訊號模板（單週期有效）

```verilog
// Purpose: one-cycle pulse to start downstream unit
always @(posedge clk) begin
    if (reset)
        unit_start <= 1'b0;
    else
        unit_start <= ((state == ST_ACTIVE) && !unit_busy);
end
```

#### 同 reg 多來源：單一 always + 優先鏈模板

```verilog
// Purpose: stream_cnt; completion resets beat progress (pattern C)
always @(posedge clk) begin
    if (reset)
        stream_cnt <= 'd0;
    else if (state == ST_IDLE)
        stream_cnt <= 'd0;
    else if (state == ST_STREAM) begin
        if (unit_done)
            stream_cnt <= 'd0;
        else if (stream_cnt == 'd0)
            stream_cnt <= 'd1;
        else if (stream_cnt <= LAST_INDEX)
            stream_cnt <= stream_cnt + 1'b1;
    end
end
```

#### 避免暫存器 Q 參與同拍清除／使能（與拆分搭配）

- **禁止** 在更新 `foo` 的 `always` 裡，用**當拍剛被 NBA 驅動的 `foo`**（或由其直接產生的組合條件）作為**同一 always 內**的清除／禁止條件；改 **延遲一拍** 的 `foo_d`／`foo_q`，或將 `foo` 僅放在**另一個** `always`。
- **禁止** 把 `state`／`next_state` 與大量 datapath、脈衝、計數器寫在同一 `posedge` block（仍遵守三段式 FSM）。

#### 跨 `always` 多驅動（拆分後必查，RTL／GLS 皆會出問題）

將大型 `posedge` block 拆成多個 `always` 後，常見遺漏：**專用 latch `always` 已驅動某 `reg`，但舊的「總控」`always` 在 `if (reset)` 或 `case` 裡仍對同一 `reg` 賦值** → 形成 **RTL multi-driven**；RTL 仿真可能仍「看似正常」，**gate-level** 常出現 **internal net 雙驅動、async clear（CN）為 X、控制脈衝失效**。

**規則（必守）**

1. **每個 `reg` 的 NBA 賦值（`<=`）只能出現在一個 `posedge clk` `always` 的 LHS**（三態／多工／`assign`+tri 等明確規格除外）。
2. **專用 shadow／latch `always`** 須包辦該訊號的 **`reset`、功能 latch、必要 default**；其他 `always` **不得**再寫此訊號。
3. 從「總控」`always` 的 **`if (reset)` 清單**刪除已外移的 `reg` 時，**勿改語意**：`reset` 仍由專用 block 清 0。
4. 修改後用 **全檔掃描**（腳本或 `rg`）確認：每個 `reg` 名稱在 `always @(posedge clk)` 內的 `<=` LHS **至多一處**。

**推薦：專用 latch `always` 模板**

```verilog
// Purpose: latch one SRAM/sample value for phase1 (sole driver of shadow_r)
always @(posedge clk) begin
    if (reset)
        shadow_r <= 16'sd0;
    else if ((state == ST_USE) && sample_en)
        shadow_r <= sram_q;   // or bus data
end
```

**反模式（須刪除總控 block 內的重複行）**

```verilog
// BAD: shadow_r driven here AND in dedicated latch always above
always @(posedge clk) begin
    if (reset) begin
        stream_cnt <= 'd0;
        shadow_r   <= 16'sd0;   // <-- remove; belongs only in latch always
    end
    // ...
end
```

**與 GLS／網表除錯的關係（通用，非 golden 路徑）**

| 現象 | 可能根因 | 建議 |
|------|----------|------|
| 控制 `reg` 在 GLS 為 X，RTL 正常 | 合成後 **同一 wire 多個 `.ZN` driver**；或 **CN/CLR 路徑** 被組合雲污染 | 在 netlist 對該 net **`grep` driver 數**；回 RTL 查 **跨 `always` 多驅動** |
| 僅改 D 端條件、GLS 不變 | 問題在 **清除／使能腳** 或 **別模組與本模組位址／控制合併** | 勿只調 functional `if`；先排除 **multi-driven** 與 **async 控制** |
| 子模組 `wgt_addr` 與 top FSM 在 netlist 合併 | 多為 **綜合合併 + 多驅動**，非 ROM 內容錯 | RTL 側：**idle 鎖死或暫存** 輸出埠；並確保 **每個 `reg` 單一 always** |

> 專案若另有 gate-level 對拍流程，細節见 `numpy-trunk-to-verilog.mdc` §10；本節只規範 **RTL 寫法**，避免產生可綜合出的多驅動網。

#### 修改後自查

- [ ] **時序與組合** `always`：每一巢狀層皆為單一 `if`／`else if`（`else`）鏈，**無**同層平行 sibling `if`
- [ ] **無**兩個以上 `posedge` `always` 對**同一 `reg`** 做 `<=`（除非三態／多工／`assign`+tri 規格已文件化）
- [ ] 專用 latch／shadow `reg`：**僅**在其專用 `always` 出現；已從「總控」`always` 的 `reset`／`case` **刪除**重複賦值
- [ ] 原 `ST_IDLE`（或 `reset`）對各控制 reg 的清零，已分到**負責該 reg 的那一個** `always`（勿在兩處各清一次）
- [ ] 脈衝類訊號每拍有 **default 無效**（或等價行為）
- [ ] `else if` 順序與註解已對照**重構前**敘述順序
- [ ] 組合 `always @(*)`：頂端 default + 無 latch；多路請求為 `if`／`else if` 互斥
- [ ] RTL 仿真與重構前一致；若需 gate-level 驗證，用**該專案**既有綜合／網表流程重跑後再比對
- [ ] （可選）對修改過的 `.v` 跑 **LHS 重複掃描**，零重複再送綜

組合仍遵守 **「避免產生 latch」**；時序與組合**共同**遵守 **「`if` / `else if` 鏈」**。本節其餘重點為拆分後的 **跨 always 多驅動**、Q 回授與 GLS 風險。

## 記憶體／ROM 讀取（通用，golden 對拍必讀）

適用所有 **合成用 ROM／SRAM macro**、由 top 匯流排供應 **read data + address** 的 datapath。不綁定特定模組名、位址切 bit 或 golden 檔名；**每個 instance 须在模組註解寫清自己的讀取契約**。

### 1. 先寫清「讀取契約」

每個使用 macro 讀取的 block，模組頭或介面旁 **必須** 用幾行文字定義（可合成 RTL 也寫；TB 與 golden 對照依此為準）：

- **位址何時有效**（例：同步 `posedge` 主時脈）。
- **讀出資料何時有效**（例：位址後第 1 拍、第 2 拍；或 falling-edge 採樣）。
- **與主時脈的關係**（例：macro `CLK` 接 `~clk`、接 `clk`、或獨立 gating）。
- **位址欄位語意**（哪些 bit 是 index／channel／bank；與軟體 flatten 順序一致）。
- 若 **同一讀取埠** 依位址區段對應 **不同語意**（權重 vs 偏置 vs 係數表），须列出 **區段與解碼規則**，不可默認 software 會猜。

未寫契約就接線 → 視為規格缺失，不可直接對 golden。

### 2. 禁止的假設（最常見對拍失敗原因）

- **禁止** 在 **更新位址的同一拍**，把 **當拍讀到的資料** 直接送進組合乘加／比較／飽和輸出（除非契約明確寫「同拍」且 golden 亦同拍）。
- **禁止** 在 **位址已指向「下一筆／下一欄／下一列」的拍**，仍用 **當拍讀出** 當作 **上一筆運算** 的係數或偏置（overlapped 流水常見 off-by-one）。
- **禁止** 把 **讀取延遲** 與 **運算延遲** 混在同一個 counter 裡卻不做對齊；須 **顯式** 區分「送位址階段」與「消耗資料階段」。
- **禁止** 模擬時假設未讀過的儲存單元為 0；未初始化在 sim 為 **X**，會污染累加器。可合成 RTL 不用 `initial` 填硬體；若僅仿真需要，用 `` `ifndef SYNTHESIS `` 包住。

### 3. 推薦做法（與契約無關的通用型態）

擇一或組合，並在註解說明與契約的對應：

| 型態 | 作法 | 適用 |
|------|------|------|
| **讀取暫存** | 位址拍 N → 資料鎖進 register → 運算只用 register | 固定 1-cycle latency |
| **地址／資料錯開** | FSM：`ADDR` 狀態只改位址；`USE` 狀態才乘加 | 任意固定 latency |
| **小型 line buffer** | 對當前 index 先掃一遍鎖一整列係數，再對輸入做 MAC | 一維權重列、係數表 |
| **顯式 pipeline** | `addr` → `data_r1` → `data_r2` → MAC | latency > 1 |

偏置／scale 若 **只依「目前輸出 index」** 而與輸入 feature index 無關：须在 **對應該 output index 的讀取契約拍** 鎖定偏置暫存，**不可** 在「已遞增 output index 的預取拍」用組合讀出的偏置。

### 4. 與軟體 golden 的關係

- 高階模型（C／Python／MATLAB 等）**預設** 常是「位址與資料同拍、無 macro 延遲」；RTL 有延遲時，要么 **RTL 對齊參考語意**（加暫存／錯拍），要么 **先改參考模型產生流程並重跑**，**不可** 兩邊時間軸不同卻直接比最終輸出。
- 對拍失敗時，**先** 比「讀取契約是否一致」與「第一個使用讀出資料的節點」，再查捨入／飽和；勿只調最終輸出。
- 模組邊界註解可註明參考向量來源（檔名、位寬、Q 格式），但 **本規則不列舉路徑**；由專案文件維護。

### 5. 除錯與驗證（通用）

- `$display` 等 log **英文**；列印 **鎖存後** 用於運算的資料，不要與 **同拍剛改位址** 的組合讀混為一談。
- 用 `$strobe` 看 **NBA 後** register 時，勿把 **本拍才賦值的輸出** 與 **組合預測值** 混比；建議同時印 **組合預期** 與 **register 輸出** 並標時序關係。
- 波形不可用時：在 **送址 / 鎖數 / 消耗** 三個邊界各設可關閉的 debug 區塊（`` `ifdef ``），預設關閉。
- **log 解讀（勿誤判 ROM 壞掉）**：
  - 匯流排上 `addr` 與 `wgt_i` 在 **送址拍** 已對、但 **buffer 檢查** 整段錯位 → 多半是 **預取索引／同拍雙寫**，不是 ROM 內容錯。
  - 逐筆 `[WGT_WR]` 與 golden 一致、`[WGT_CHK]` 卻左移一格 → 典型 **slot0 被寫兩次** 或 **寫入 index 與遞增脫鉤**。
  - 同一 `` `ifdef `` 下 **多個同型子模組**（例：QKV / PROJ 各一個 `linear`）都列印 → 可能把 **PROJ 權重** 當成 QKV golden；须 **參數或階層** 只開一個 instance 的 dump。

### 6. ROM 初值與 RTL 修改範圍

- 係數 **內容** 來自綜合前之 memory 映像；**僅改讀取時序／暫存／FSM** 通常 **不必** 重產映像。
- **若** 變更 flatten 順序、位寬、深度、表內容 → 须重產映像並在回覆中告知（細節见專案 golden／ROM 流程規則，本節不列檔案表）。

### 7. 專案範例：ROM `CLK(~clk)` 與權重列預取（可選參考）

> **§7.1–§7.6** 為本 repo **SGLATrack / CARE backbone** 除錯沉澱（權重 ROM、`linear` 預取）。**其他專案**可略過或替換為自己的 ROM 契約；**§7.7** 為 **通用 1P SRAM** 規則，建議所有使用 macro 的專案遵守。

細部 golden 路徑、ROM 檔名见 `numpy-trunk-to-verilog.mdc`（若適用）。

#### 7.1 讀取契約（須寫在模組頭）

- 本專案 weight ROM macro 多為 **`CLK(~clk)`**（位址在 **主時脈 posedge** 送出，**讀出 `wgt_i`/`bias_i` 在下一拍 posedge** 才對應「上一拍所送位址」）。
- 建議在模組頭固定寫成三行契約，例如：
  - `posedge T`：送 `w_addr`（feat / neuron 由當拍 index 決定）。
  - `posedge T+1`：`w_i` = 位址在 `T` 所指定之係數；**此拍** 才可寫入 `wgt_buf[k]`。
- **禁止** 把「送位址拍」的組合 `w_i` 直接當 MAC 乘數（除非契約明訂同拍且 golden 同拍）。

#### 7.2 推薦：兩拍 `stream` / `stream_r` 預取一整列權重

對「每個 output neuron 先鎖定 32 個 weight，再對 32 個 activation 做 MAC」之 **linear / 內嵌 QKV**：

| 拍 | 條件 | 動作 |
|----|------|------|
| 送址 | `stream=1` | `w_addr` 使用 **當前** `feat`（尚未遞增） |
| 鎖數 | `stream_r=1`（`stream` 延遲 1 拍） | `wgt_buf[feat] <= w_i`，**然後** `feat++`（最後一筆寫完再進 MAC） |

- 與 `verilog_backbone/care_attention.v` 之 `rp_feat_mac` + `stream_r` 寫法 **等價**；`verilog_backbone2/linear.v` 用 `wpre_feat` 同名語意即可。
- **最後一筆 weight**（`feat == IN_DIM-1` 且 `stream_r`）：寫入 `wgt_buf[31]` 後關閉 `stream`，並在進入 MAC 前清 `acc` / `mac_idx`（勿在 MAC 狀態 **每拍** 清零累加器）。

#### 7.3 偏置鎖定（勿與權重送址拍混淆）

- 偏置只依 **output neuron**，與 input feat 無關。
- **推薦**：在 **第一筆 weight 寫入** 時鎖定：`state==WPRE && stream_r && feat==0`（與 `qkv_bias_ce` / `proj_bias_ce` 同拍）。
- **禁止** 僅在 `stream` 上升沿鎖 bias 卻未對齊「上一拍已送 `neu*IN_DIM+0` 位址」——易拿到 **上一 neuron** 或 **錯區段** 的 `bias_i`。
- **禁止** 在 `feat` 已遞增、位址已指向下一係數的拍，用組合 `bias_i` 當作當前 neuron 的 bias。

#### 7.4 位址解碼與匯流排（ROM 沒壞也常栽在這裡）

- 矩陣列為 **`[out_neuron][in_feat]`** 時：flatten 位址須 **`neuron * IN_DIM + feat`**；**proj** 等以 neuron 為列索引時，勿把 `local[4:0]` 當 neuron、`local[9:5]` 當 feat **接反**。
- **`wgt_addr` 拼接後位寬** 須檢查 **type 欄位**（如 `wtype=3'b010`）是否被截斷；concat 後若只取低 16 bit，會讀到 **錯誤 ROM 區**（例：attn 變 fc1）。
- 子模組 **`start` 與第一筆 `x_valid` / `stream`**：若 `start` 與資料同拍或晚一拍，會 **漏掉 index 0** 的輸入或權重；controller 須在規格註明「誰先誰後」。

#### 7.5 明確禁止的反模式（曾造成 ~10240/10240 QKV 全錯）

- **禁止** 另用 `wr_slot`，並在 **`stream && stream_r` 同拍** 把 `wr_slot` 清 0 → **slot0 寫兩次**（`ffe2` 被 `002f` 覆蓋，buffer 左移）。
- **禁止** 在 `stream` 上升沿與 `stream_r` 同拍 **各寫一次** `wgt_buf[0]`。
- **禁止** 在 MAC 累加狀態 **每個 posedge** 將 `acc` / `mac_feat` 清零（僅允許在 **預取結束 → MAC 開始** 的邊界清一次）。
- **禁止** 用 `{1'b0, term}` 對 Q8.8 負數做零擴展再累加（须 **符號延伸**）；見 CARE `kv` / `attn` 路徑除錯紀錄。

#### 7.6 除錯 macro 慣例

- 權重預取 dump（例：`DUMP_WGT_PREFETCH`）應以 **module parameter**（例：`DUMP_WGT=1`）限制 **單一 instance**（通常只開 QKV `linear`），避免與 PROJ 混淆。
- `$display` 標籤建議區分 **`[WGT_WR]`**（當拍寫入）與 **`[WGT_CHK]`**（進 MAC 前 buffer 快照）；兩者不一致時 **先查 7.2／7.5**，勿先重產 ROM。

#### 7.7 SRAM macro（1P、`CLK(clk)`）讀寫 cycle 對拍

**專案硬性契約（必守）**：所有 ROM/SRAM macro 的 **`.CLK` 一律接 `clk` 本身（不是 `~clk`）**，位址取樣與讀/寫都在 **`posedge clk`**，**不得在 `negedge` 做任何動作**（包含 TB 的 SRAM 模型）。下文 cycle 表以此為準。

本節為 **activation／feature map 用 TSMC Single-Port SRAM** 之必守時序；與 §7.1 ROM 契約並列。若 compiler 產出為 **Dual-Port** 或 `CLK` 接法不同，須在模組頭 **改寫契約**，不可默認與本節相同。

##### 7.7.1 介面與 port 型態

- 本專案 SRAM macro 預設為 **Single Port（1P）**：每 cycle 僅 **read 或 write 擇一**，由 **`WEB`** 區分；**不可** 同址同拍 read+write。
- 常見控制（active-low）：**`CEB=0`** 啟用 macro；**`WEB=0`** 寫入、**`WEB=1`** 讀取。
- **`CEBM`／`WEBM` 等 B-port 腳位未接** → 視為 1P，勿當 2P 同拍讀寫。
- 多顆 macro（如 `Sram_fmap1~4`）並行 = **多顆 1P 同時讀／寫不同 bank**，不是一顆 dual-port。

##### 7.7.2 讀取契約（須寫在模組頭，與 §1 一致）

本專案 1P SRAM 一律 **`CLK(clk)`**（controller 與 macro 同 **`posedge clk`**）。macro 在 **posedge 取樣位址**、輸出資料**暫存於 posedge**，故「**決定位址的那拍 → 可用資料**」**自然是 2 個 posedge**（macro 的 1 拍 read latency 疊在「位址送出」那拍之上）：

| 拍 | 動作 |
|----|------|
| **`posedge T`** | 位址 `A`（由 reg 或當拍穩定的組合值）成形、`CEB=0`、`WEB=1`（read） |
| **`posedge T+1`** | macro 於此 posedge 取樣 `A@T`，`Q` 暫存輸出 |
| **`posedge T+2`** | **`Q` 對應 `A@T` 之資料有效**；**此拍** 才可鎖進 `data_r` 或送 MAC |

- **每拍一筆的串流讀**（如 MAC/ACCUM/NORM）：每拍發一個位址、資料**晚 2 拍**到，pipeline fill +2，吞吐仍 1 筆/拍；消費端窗口要相對位址 **+1 對齊**（相較舊 `~clk` 1 拍模型）。
- **2-phase 預載讀**（phase0 發址 / phase1 取值，如權重/bias/gamma/beta ROM）：phase 本身已有一拍寬裕，posedge macro 的 `Q` 剛好落在 phase1 → **capture 仍在 phase1 結束拍**，**不需額外 retime**。
- **TB 的 SRAM 模型也必須 posedge**：`always @(posedge clk) q_reg <= MEM[addr];`（registered-data），**禁止** `@(negedge clk)` latch 位址。

建議模組頭固定三行（依 instance 改信號名）：

```verilog
// SRAM read contract (1P, CLK = clk, posedge-sampled):
//   posedge T:   sram_addr stable, CEB=0, WEB=1
//   posedge T+2: sram_q valid for addr@T; use only latched data_r (or via ADDR/USE), never addr@T's same-cycle Q
```

- **禁止** 在 **`posedge T` 更新 addr 的同一拍**，把組合 **`Q`** 直接送進乘加／比較（除非 datasheet＋golden 明訂 0-cycle，且 TB 同拍）。
- 若 datapath 需再多一拍（例：`data_r <= sram_q` 後才 MAC），須在契約寫 **總 latency = 2**，並對齊 FSM counter；參考 `reference/rongxuan_verilog/CFVT.v` 之 `temp_fm1_out <= fm1_out`、`temp_ps_out <= ps_out`。

##### 7.7.3 寫入契約

預設 **write 占 1 個 `posedge clk` cycle**（與 read 相同，不可與 read 同拍混用同一 1P macro）：

| 拍 | 動作 |
|----|------|
| **`posedge T`** | **`A`**、**`D`** 穩定；**`CEB=0`**、**`WEB=0`** |
| **macro `negedge clk`** | 寫入完成 |

- **位址遞增** 應在 **寫入拍之後** 的獨立拍進行（勿在 **`WEB` 仍為 write 的同一拍** 用「下一筆 addr」當作剛寫入資料之 index）；典型：`WEB=0` 拍寫入 → 下一拍 `A++`。
- **禁止** 寫入完成當拍立刻 read 同址並假設 `Q` 為新值（1P macro 無 bypass；須至少隔 **1 read cycle** 或改 schedule）。

##### 7.7.3.1 Producer 邊界：`reg valid` + 組合 `data`／`addr`（必守）

部分子模組（典型 **`layer_norm` S_NORM**）在同一 `posedge` 內：

- **`y_sat_o`（或等價 comb 輸出）** 與 **`feat_addr_o`** 對齊當拍資料；
- **`y_valid`** 為 **reg**，在該 posedge **NBA** 才變 1，且 **`addr` 同拍遞增**。

Consumer（top／controller）若用 **組合 `if (y_valid)`** 驅動 SRAM **`WEB=0`**，並以 **當拍 `feat_addr_o`** 當寫入 index → **off-by-one**：**index 0 漏寫**、後續整段錯位；與 macro read latency 無關，是 **握手錯**。

**推薦 capture 契約**（須在 consumer 模組頭或 SRAM mux 旁註明）：

| 拍 | 動作 |
|----|------|
| **`posedge T`**（producer **`out_beat_o` 或 `state==S_NORM` 為 1**） | 鎖 `{flat, y_sat_o}` 進 **`s1_wr_flat_lat` / `s1_wr_din_lat`**（`flat = tok×C + feat_addr`，row-major） |
| **`posedge T+1`**（**`s1_wr_do`** 為 1） | 驅動 **`A/D/WEB=0/CEB=0`** 寫入 macro；**`USE_REG_BUF`** 路徑同拍 **`out_buf[lat] <= din`** |

- **禁止** 以 **`y_valid` 組合** 當 SRAM 寫入 enable（除非已證明 producer 的 `valid` 與 `data`／index **同拍穩定**，見下）。
- **`y_o`（reg）** 比 **`y_sat_o`（comb）** 晚一拍；capture 以 **`y_sat_o` + 當拍 `feat_addr_o`** 為準，勿混用。
- Producer 須在 port 註解或 **`out_beat_o`** 標示可 capture 的 beat（見 `layer_norm.v`）。

**與 stream 寫入的區別**（勿套用同一條件）：

| 來源 | 握手 | SRAM 寫入 |
|------|------|-----------|
| **`tb_y_valid` + `tok_wr_ptr`** 等 | `valid`、`data`、index **同拍穩定**；ptr **NBA** 遞增 | 組合 **`if (valid)`** 寫入 **通常 OK** |
| **`layer_norm` → backbone norm** | `y_sat_o`／`feat_addr` comb；**`y_valid` reg 晚一拍** | **須 posedge latch + 延遲一拍寫**（上表） |

##### 7.7.4 推薦 RTL 型態（擇一，註解對應契約）

| 型態 | 作法 | 總 read latency |
|------|------|-----------------|
| **直接鎖 `Q`** | `posedge T` 送 addr；`posedge T+1`：`buf <= sram_q` | 1 |
| **顯式 `data_r`** | 同上，MAC **只** 用 `data_r` | 1（消耗在 T+2 若 MAC 再晚一拍） |
| **ADDR／USE 兩段** | FSM：`S_ADDR` 只改 addr；`S_USE` 才乘加 | 依 FSM 固定 |
| **line／window buffer** | 連續多 addr 讀入 shift register 再 conv | 首元素 1 cycle，其後可 overlap |

##### 7.7.5 與 golden 對拍

- numpy／Python check **預設無 macro 延遲**；RTL 有 **T→T+1** read 時，須 **加暫存／錯拍** 或 **更新 golden 產生流程**（見 `numpy-trunk-to-verilog.mdc`），不可兩邊時間軸不同卻比最終 activation。
- 除錯時依 **§16 前向對拍**：先確認 **第一個使用 `Q` 的節點** 與契約一致，再查量化。

##### 7.7.6 明確禁止的反模式

- **禁止** 1P macro 同拍 **`WEB=0` 寫入** 又讀 **`Q`** 做 MAC。
- **禁止** `conv_cnt`（或同類 counter）在 **送 addr 拍** 與 **消耗 `Q` 拍** 使用 **同一計數值** 卻未錯開（off-by-one 窗口／權重）。
- **禁止** 未寫 §7.7.2 契約就接 `Sram_*`；綜合能過但 golden 全錯。
- **禁止** 對 **`layer_norm` 類 producer** 用 **`y_valid` 組合** 寫 SRAM，並以 **同拍 `feat_addr_o`** 當 **`A`**（见 §7.7.3.1）；典型症狀：**`flat=0` 未寫**、**`first_bad_idx=0`**、sim 中 **`s1_q@0` 為 X**。
- **禁止** 因 **depth 相同** 就把 **不同語意／不同 flatten 公式** 的 activation 塞進 **同一顆 1P SRAM**（见 §7.7.7）。
- **`RTSEL`／`WTSEL`** 若與 compiler 默認不同，須在模組頭注明 read/write cycle 是否仍為 1；變更時重新對拍。

##### 7.7.7 同 depth、不同 flatten — 禁止共用一顆 1P SRAM

當兩條 datapath **有效地址範圍相同**（例如皆為 `0 .. N*C-1`），但 **index 公式不同**（不同 tensor layout），**同一 flat index 可代表不同語意** 的資料。

- **禁止** 假設「10240 深度夠大就可共用一顆 scratch SRAM」；**寫入 A 語意** 可能 **覆蓋 B 語意** 仍須讀取的元素（即使 index 數字相同）。
- **症狀**：中間層 golden **部分 index 對、部分錯**；常見 **約一半 FAIL**（alias 週期性出現）。
- **解法（擇一）**：
  - **分離 macro**（各 layout 一顆）；
  - **統一 flatten**（軟體與硬體同一公式）；
  - **時間分割**（絕不 overlap 讀寫同一 layout 的生命週期）；
  - **parent 單一 scratch + 子模組 port 讀**（子模組不再 `S_LOAD_X` 複製），並在 FSM 上保證 **覆寫前讀完**。

模組頭應寫清 **每種 layout 的 flatten 公式** 與 **哪顆 SRAM 承載哪種 layout**。

##### 7.7.8 子模組向 parent 請求 activation read（port + 2-phase）

當子模組 **不再持有** 大型 `x_in_buf`，改由 **parent 的 scratch SRAM / reg** 供料時，建議介面（名稱可改，語意固定）：

| 方向 | 訊號 | 說明 |
|------|------|------|
| child → parent | `*_rd_en` | **ADDR 拍** 為 1：請 parent 對 macro 送 read |
| child → parent | `*_rd_flat` | token-major（或契約訂定）flat index |
| parent → child | `*_x` | **USE 拍** 有效資料（`USE_REG_BUF`：comb `buf[flat]`；SRAM：`Q` @ T+1） |

**時序（與 §7.7.2 一致）**：

| 拍 | child | parent SRAM mux |
|----|-------|-----------------|
| T | `*_rd_en=1`，送出 `*_rd_flat` | `WEB=1`，`A=flat` |
| T+1 | `*_rd_en=0`，鎖 `*_x <=` 讀出資料 | 消耗 `Q` |

**禁止** 在 child 內用 **大 2D shadow reg** 取代 parent SRAM（见專案 `SRAM_suggestion.mdc` §5.1 #2）。

**parent 多路請求**（norm capture 寫、子模組 FC1 讀、stream 寫出）須在 **combinational mux** 定義 **固定優先級**，並在 parent 模組頭列出；**禁止** 兩路同拍對 **同一 1P** 又讀又寫。

##### 7.7.9 `ADDR`／`USE` phase 命名慣例（FSM）

- 使用 **`_phase`、`_rd_phase`、`feed_phase`、`qkv_x_phase`** 等 **1-bit**：`0` = ADDR（送址），`1` = USE（消耗資料／寫回）。
- **in-place read-modify-write** on 1P SRAM：**必須** 拆兩拍（或 shadow scalar + 寫回拍），不可依賴 reg 陣列「同拍讀舊寫新」語意。
- 模組頭 **契約三行**（§7.7.2）須標明本模組採用哪種 phase 名稱。

##### 7.7.10 輸出埠與 STA（`assign y_o = sram_q`）

- **禁止** 在未評估 STA 前，把 **top-level output** 直接 **`assign` 到 SRAM `Q`**（例：`assign y_o = s1_q`）：合成會出現 **`SRAM CLK → pad`** 組合 critical path，且與 **`set_output_delay`** 疊加後易違反 setup。
- **推薦**：`S_OUT` 在 **`_phase==1`（USE）** 鎖 `y_o_reg <= sram_q`，`y_valid` 延後一拍對齊；或接受較鬆的 output delay 並在 SDC 單獨約束 `y_o`。
- 功能語意仍以 **§7.7.2 兩拍 read** 為準：phase0 送 `A`，phase1 消耗 `Q`。

> **Reg → SRAM 逐步改寫**：见下方 **§8**（檢查清單、port mux 模板、常見錯誤對照）。

---

## §8 Reg 陣列改 SRAM macro（實作檢查清單，必守）

將 `reg [15:0] buf [0:DEPTH-1]` 換成 **1P SRAM macro**（常見 `CLK(~clk)`）時，**不可** 只刪 `reg`、接 instance 就結束。須依本節與 **§7.7** 同步改 **FSM、port mux、握手、flatten 註解**。Instance 規劃、參考向量路徑由**該專案**另文維護。

### 8.1 改前先填的規格表（寫在模組頭）

每顆由 `reg` 改成的 SRAM，模組頭 **至少** 列出：

| 欄位 | 內容 |
|------|------|
| Legacy 名稱 | 例：`x_buf`、`tmp_buf`、`sh1_buf` |
| Macro 型號 | 例：`Sram_tok1`、`Sram_x`（**compiler 產物檔名**，不可混用） |
| depth × width | 例：10240×16；**addr 位寬** 須與 macro `A` 一致 |
| Flatten 公式 | 例：`tok*C+feat`（token-major）或 head-major 公式 |
| 生命週期 | 誰寫、誰讀、何 state 重疊；能否與另一 buffer **時間多工** |
| 讀 latency | 預設 **1 posedge**（§7.7.2）；消耗在 ADDR+1 或 +2 |
| Port mux 優先級 | 多路讀寫時 **固定順序**（见 §8.4） |

**禁止** 僅因 depth 相同就共用一顆 SRAM（不同 flatten → index alias，见 §7.7.7）。

### 8.2 改寫順序（建議一步一步對拍）

1. **盤點** 所有 `reg [...] buf [...]` 與其 **讀寫 always／assign**（含子模組 shadow copy）。
2. **模組頭** 寫 §8.1 表格 + §7.7.2 契約三行。
3. **刪除** legacy `reg` 陣列宣告與對應的「單拍 `buf[idx]`」讀寫。
4. **例化 macro**（§8.3），**勿** 在 `always` 內宣告控制 `reg`。
5. **新增** port mux `always @(*)`（§8.4）：預設 **idle = 不選通**（`CEB=1` 或 `CEB=0, WEB=1` 依專案慣例，**整檔一致**）。
6. **FSM**：凡原「同拍讀 `buf[i]` 做運算」→ 改 **ADDR phase + USE phase**（§8.5）；寫入改 **latch 一拍再寫**（§8.6）若 producer 為 `layer_norm`。
7. **子模組** 改 **parent port 請求讀**（§7.7.8），**禁止** 在 child 再留 10240 深 shadow `reg`。
8. **VCS**：編譯列 **列入每一種** `Sram_*.v`（同 depth 亦可能 **不同檔名**，见 `SRAM_suggestion.mdc` §1.0）。
9. **對拍**：先比 **第一個使用讀出資料的節點**，再比最終輸出（§16 `numpy-trunk-to-verilog`）。

### 8.3 Macro 例化模板（1P、`CLK(~clk)`）

```verilog
// SRAM read contract (1P, CLK = ~clk): see verilog_rule.mdc §7.7.2
//   Layout: flat = tok * EMBED_DIM + feat   (token-major)
//   Roles: norm capture write; S_OUT stream read (mutually exclusive via mux)

Sram_tok1 u_sram_x (
    .CLK   (~clk),
    .CEB   (s_ceb),      // 0 = macro enabled
    .WEB   (s_web),      // 0 = write, 1 = read
    .A     (s_addr),
    .D     (s_din),
    .Q     (s_q),
    .CEBM  (),
    .WEBM  (),
    .BWEB  (16'b0),
    .BWEBM (16'b0),
    .RTSEL (2'b00),      // 與 compiler 默認一致；變更須重對拍
    .WTSEL (2'b00)
);
```

**必查**：

- **`CEBM`／`WEBM` 未用** → 1P，**禁止** 同拍 read+write。
- **`BWEB`**：全寫入時 `16'b0`；partial write 須與 macro 規格一致。
- **addr／din 控制 `reg`** 在 module 頂層；**`Q` 只讀**，勿在送址同拍當 MAC 乘數（§7.7.2）。
- 同一設計內 **`Sram_tok1` 與 `Sram_x` 不可只編譯一種 `.v` 檔** 取代所有 instance。

### 8.4 Port mux（`always @(*)`）必守寫法

對 **每一顆** 1P SRAM 獨立一個 mux block（或明確分區），**區塊最上方先給 idle**：

```verilog
always @(*) begin
    s_ceb  = 1'b1;   // disabled (active-low); match macro datasheet / idle convention
    s_web  = 1'b1;   // read mode default (safe idle)
    s_addr = {14{1'b0}};
    s_din  = 16'd0;

    if (capture_wr_pulse) begin
        s_ceb  = 1'b0;
        s_web  = 1'b0;
        s_addr = wr_flat_lat;
        s_din  = wr_din_lat;
    end else if (stream_rd_addr_phase) begin
        s_ceb  = 1'b0;
        s_web  = 1'b1;
        s_addr = rd_addr;
    end
    // else: idle — 禁止漏寫導致 latch（见「避免產生 latch」）
end
```

| 規則 | 說明 |
|------|------|
| **互斥** | 同一拍 **最多一路** 驅動 `CEB/WEB/A/D` |
| **優先級** | 寫入 capture **高於** 背景讀；**寫與讀不可同拍** |
| **列出順序** | 在模組頭註解 **1→2→3**（例：`tmp_wr_do` > `mlp_norm_rd` > `attn_feed`） |
| **禁止** | 同層兩個 `if` 可能同時成立卻未 `else if` → 多驅動或 X（亦違反共通 **「`if` / `else if` 鏈」**） |

參考：top 層對單顆 1P SRAM 的 **port mux**（多路請求 `if`／`else if` 互斥、區塊頂端 idle）。

### 8.5 Reg 讀寫 → 2-phase FSM 對照

| 原 `reg` 寫法 | SRAM 改法 |
|---------------|-----------|
| `buf[i] <= din`（單拍寫） | `posedge`：`WEB=0`，`A=i`，`D=din`；**下一拍** 才可 `i++` |
| `x = buf[i]`（同拍讀+算） | **phase0**：`WEB=1`，`A=i`；**phase1**：`x_r <= Q`，MAC 只用 `x_r` |
| `for` 掃整列再 MAC | **stream**：每元素 2 phase，或 line buffer（§7.7.4） |
| 同址讀→改→寫回 | **兩拍** 或 scalar shadow + 寫回拍（§7.7.9）；**禁止** 1P 同拍 RMW |

**Stream 讀出範本**（`S_OUT`，`out_rd_phase`）：

| phase | `out_rd_phase` | SRAM | 消費端 |
|-------|----------------|------|--------|
| ADDR | 0 | `WEB=1`，`A=out_rd_addr` | 不送 `y_valid` |
| USE | 1 | 不變 addr | `y_valid=1`，資料來自 **`Q`（上一拍 addr）** |

時序 `always` 範例（2-phase 讀出）：

```verilog
// phase0: hold addr for negedge macro sample; phase1: consume Q, then advance addr
if (out_rd_phase == 1'b0)
    out_rd_phase <= 1'b1;
else begin
    if (out_rd_addr < LAST)
        out_rd_addr <= out_rd_addr + 1'b1;
    out_rd_phase <= 1'b0;
end
```

**禁止** 在 `out_rd_phase==0` 同拍 `assign y_valid=1` 並用當拍 `Q`（off-by-one）。

### 8.6 Producer 寫入：`layer_norm` / `out_beat_o`（§7.7.3.1）

| 拍 | 動作 |
|----|------|
| T | `out_beat_o==1`：鎖 `wr_flat_lat`、`wr_din_lat`（用 **`y_sat_o` + `feat_addr_o`**，勿用晚一拍的 `y_valid`） |
| T+1 | `wr_do==1`：mux 驅動 `WEB=0` 寫入 macro |

**禁止** `if (y_valid)` 組合寫 SRAM 並用同拍 `feat_addr_o` → **index 0 漏寫**、後段全錯。

### 8.7 子模組向 parent 請求讀（取代本地 `x_in_buf`）

| 拍 | child | parent |
|----|-------|--------|
| T | `norm_rd_en=1`，`norm_rd_flat` | `WEB=1`，`A=flat` |
| T+1 | `norm_rd_en=0`，`norm_x <= Q` | MAC／linear 用 `norm_x` |

**禁止** 保留 `S_LOAD_X` 10240 拍複製，若 parent 已有同 layout SRAM（见 `SRAM_suggestion.mdc` Phase 3 目標）。

### 8.8 時間多工（同一顆 SRAM 多種語意）

僅在 **生命週期不重疊** 時允許（例：`v_buf` 與 `ao_buf` 共用 `Sram_v`）：

- 模組頭 **Deviation** 區塊寫清：哪個 state 寫、哪個 state 讀、為何不重疊。
- **禁止** 僅為省面積把 **不同 flatten** 塞同一顆 macro（§7.7.7）。

### 8.9 改完必跑的自查（仿真前）

- [ ] 模組頭有 **flatten 公式** + **mux 優先級** + **§7.7.2 三行契約**
- [ ] 無殘留 `` `ifdef USE_REG_BUF `` 與 **未刪淨的大 `reg` 陣列**（若專案已 SRAM-only，勿留死碼）
- [ ] 每顆 SRAM 的 mux **idle 預設** 已寫、**無組合 latch**
- [ ] 所有「讀 `buf[i]` 運算」已改 **USE phase** 或 `*_r` 暫存
- [ ] `layer_norm` 寫入走 **lat + `wr_do`**，非 `y_valid` 組合寫
- [ ] VCS filelist 含 **所有** 用到的 `Sram_*.v`
- [ ] Top output 若 `assign y_o=Q`，已知 STA 風險（§7.7.10）

### 8.10 常見錯誤症狀 → 先查哪一條

| 症狀 | 最可能原因 | 對應節 |
|------|------------|--------|
| `first_bad_idx=0`、flat0 未寫 | `y_valid` 組合寫 SRAM | §7.7.3.1、§8.6 |
| golden **約一半** FAIL、其餘對 | 不同 flatten 共用一顆 SRAM | §7.7.7 |
| QKV／權重 **整段錯位一格** | ADDR/USE 同拍消耗 `Q`；或 `stream` 雙寫 slot0 | §7.7.2、§7.5 |
| sim 中 `Q@0` 為 X | 未寫入就讀；或 idle 時 `CEB` 語意錯 | §8.4 |
| 合成過、對拍全錯 | 時間軸與 Python 不同，未加暫存 | §7.7.5 |
| STA：`SRAM CLK → y_o` 違反 | 組合輸出 + `set_output_delay` 過緊 | §7.7.10 |
| elaboration 找不到 module | 少編譯 `Sram_x.v` 等 | §8.2 step 8 |

### 8.11 與其他規則的關係

- **ROM 權重預取**（`stream`／`stream_r`）：仍用 **§7.1–§7.6**，與 activation SRAM **分開** 處理。
- **改 SRAM 不必重產 ROM**；**flatten／depth／macro 型號變更** 須重產映像（§7.6、§20 `numpy-trunk-to-verilog`）。
- 面積／instance 規劃、head conv buffer：见 **`SRAM_suggestion.mdc`**。

## 驗證
- 測試平台用 Verilog 撰寫，DUT 必須保持可合成
- 包含 clock 產生與 reset 行為
- 與參考模型（testbench、軟體 golden、固定向量）做 bit-accurate 比對（若專案要求）
- **Simulation 列印文字必須為英文**：`$display`、`$strobe`、`$fdisplay`、`$fwrite`、`$monitor` 等**執行時會輸出到 log／終端機的字串**一律使用 **ASCII 英文**（可含數字與常見標點），**禁止中文**；使用者 VCS 執行環境不支援中文輸出。模組內 `//` 註解仍可用中文，但不得把中文寫進上述 system task 的字串參數。

---
> Source: [whalefine/s3lab_research_v2](https://github.com/whalefine/s3lab_research_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
