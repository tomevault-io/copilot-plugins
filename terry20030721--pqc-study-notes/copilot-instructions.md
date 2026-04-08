## pqc-study-notes

> 在探討記憶體怎麼防撞車之前，必須先看懂整顆晶片的「資料管線 (Data Pipeline)」。

# 🏆 終極解碼筆記：CFMMS 零衝突記憶體架構與 Radix-4 矽晶片實作

## 📍 總覽：晶片的心臟、大腦與神經網路 (Figure 4 ~ 6)

在探討記憶體怎麼防撞車之前，必須先看懂整顆晶片的「資料管線 (Data Pipeline)」。

### 1. Figure 4：Top-Level Architecture (晶片頂層工廠)
定義了資料的生命週期：
* **SRAM Banks (左右倉庫)**：根據 $R \times d$ 切割出的實體記憶體，資料從這裡出發，算完後「原地寫回 (In-place Write-back)」。
* **CFMMS (神經網路/排線)**：夾在倉庫與機台之間，負責把「邏輯位址」瞬間翻譯成「不塞車的實體 Bank 編號」。
* **Radix-4 BFU (核心機台)**：一口氣吞下 4 筆資料的重型運算單元。

### 2. Figure 5：BFU Microarchitecture (蝴蝶單元微架構)
* **Barrett Reduction 的救贖**：資料進來後先乘上旋轉因子 $\omega$。為了避免傳統除法器 $\pmod{q}$ 拖垮晶片，採用 Barrett Reduction 將除法轉為「乘法與位移 (Right Shift)」，在極短週期內消化完畢。
* **交叉加減網路**：透過兩層加減法器網路，榨取數學對稱性，免費產生 4 筆新資料。

### 3. Figure 6：Address Generation Unit, AGU (大腦指揮官)
* **任務**：產生餵給 CFMMS 的「邏輯位址」。
* **硬體魔法**：不使用加法器算跨距，而是利用計數器搭配**「線路左移 (Left Shift) 與拼接」**，瞬間吐出 $0, 64, 128, 192$ 等隨 Stage 變化的抓取位址。

---

## 📍 核心篇：4.1 理論與公式解碼 (Equations & Algorithm 4)

演算法目標：將邏輯位址 $a$ 轉換成實體 Bank 編號 ($BI$) 與 Bank 內部地址 ($BA$)。

### 📜 關鍵數學公式終極翻譯

| 公式編號 | 數學表達式 | 物理意義與硬體實現 |
| :--- | :--- | :--- |
| **公式 1** | $a = a_H \cdot (R \cdot d) + a_M \cdot R + a_L$ | **位址切片**：將二進位邏輯位址剪成幾段。$a_L$ 是原始意願，$a_H$ 是深度特徵。 |
| **公式 2** | $c = (\sum a_{H,i}) \pmod{R}$ | **陣營標籤 (算命仙)**：即 Algorithm 4 的 Coresidual。在硬體上透過 **XOR 閘** 實現，利用高位元差異抽出標籤 $c$。 |
| **公式 3** | $BI = (a_L + c) \pmod{R}$ | **實體 Bank 編號 (防撞咒語)**：若標籤 $c$ 觸發，則翻轉 $a_L$（踩到 NOT 閘），強制將撞車資料踢往其他 Bank。 |
| **公式 4** | $BA = a_H$ | **實體 Bank 深度**：直接拿高位元當作格子位址，完全不需重算。 |

### 📊 Figure 7：防撞車地圖解析
* **直排不是 Stage**：圖 7 的 4 個直排是 $BI = \pi_c(a_L)$ 排列函數的 4 種陣型。
* 💡 **【除錯：科學怪人路線】**：你抓出的撞車路徑 `01 -> 67 -> 45 -> 67` 證實了：在同一個 Clock Cycle，狀態機是全局鎖定的。不能第一步走斜線，第二步走直線。死守單一公式走到底，數字絕對完美錯開。
* 💡 **【除錯：$d=4$ 的財富自由】**：當總 Bank 數 ($R \times d = 16$) 滿溢時，$BI = \pi_c(a_L)$ 不需要翻轉，線路極致無腦化。

---

## 📍 實踐篇：4.2 降維打擊的硬體架構 (Figure 8)

作者證明數學公式雖雜，但化成電路後廉價到不可思議：

1.  **MACC 單元 (公式 2 化身)**：拋棄加法器，僅用 **XOR 閘** 算完 $\sum a_H \pmod{R}$。
2.  **PM 單元 (公式 3 化身)**：拋棄除法器，僅用 **MUX (多工器)** 搭配 **NOT 閘 (反閘)**。
3.  **Strategy 0**：捨棄耗時的 OR/XNOR 閘，用最乾淨的 XOR → MUX 達到「最輕量化 (Lightweight)」。

> [!IMPORTANT]
> **💡 【除錯紀錄：血淋淋的 6-bit 撞車實錄】**
> 帶入 $N=64, d=2$ 的真實位址：`0, 4, 8, 12` (尾巴全是 `00`)：
> 1. **高位元不同**：分別為 `000, 000, 001, 001`。
> 2. **MACC (XOR) 顯靈**：算出控制訊號 $c$ 分別為 `0, 0, 1, 1`。
> 3. **PM (MUX + NOT) 強制分流**：`0, 4` 走原路；`8, 12` 因 $c=1$ 踩到 NOT 閘，低位元翻轉，被踢去 Bank 2 與 6。
> **結論**：利用高位元算命，強制翻轉低位元。

---

## 📍 實戰篇：4.3 Dilithium 的完美殺戮 (Case Study)

對準後量子密碼學標準 Dilithium 的最終武力展示：

* **天選之數 (N=256)**：長度剛好是 $4^4$。Radix-4 BFU 僅需 4 個 Stage 即收工，碾壓 Radix-2 的 8 個 Stage。
* **無狀態的 In-place 與 INTT 魔法**：
    * 讀與寫共用公式，寫入時算去哪，讀取時就從哪拿，不需查表。
    * 因 NTT 規律的數學對稱性，防撞 MUX 網路可直接借給 **INTT** 用，零額外硬體成本。
* **整體吞吐量封頂**：CFMMS 保證「每回合必給 4 筆資料」，Barrett Reduction 保證「拿到瞬間算完」，達成 100% Pipeline 利用率。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Terry20030721)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Terry20030721)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
