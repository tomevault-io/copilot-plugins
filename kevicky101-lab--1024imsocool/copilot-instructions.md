## 1024imsocool

> 這個資料夾記錄了 `biz_1024` 資料庫（SQL Server）結構還原與 ER Diagram 產出的過程與成果。

# biz_1024 ER Diagram Project

這個資料夾記錄了 `biz_1024` 資料庫（SQL Server）結構還原與 ER Diagram 產出的過程與成果。

## 背景

`biz_1024` 是從既有資料庫 `biz00` 用 `SELECT INTO` 複製出來的一份副本。`SELECT INTO`
只會複製欄位定義與資料，**不會複製 Primary Key / Foreign Key / Index / Default 等約束**，
所以複製完成當下 `biz_1024` 裡的每一張表都沒有任何 PK 或 FK。

## 資料庫連線資訊

- Server: `163.17.141.61,8082`（SQL Server，逗號後面是連接埠）
- Database: `biz_1024`
- Login: `nutc`
- Password: 不寫在任何檔案裡（含此檔），請自行以環境變數或密碼管理工具保存，
  勿以明碼提交到版本控制。

## 這個資料夾裡有什麼

| 檔案 | 說明 |
|---|---|
| [er_diagram.html](er_diagram.html) | 整個資料庫的 E-R Diagram（純 HTML/SVG，瀏覽器打開即可看） |
| [add_constraints.sql](add_constraints.sql) | 補齊 PK/FK 的 `ALTER TABLE` 腳本。**僅產生、未執行**，需要你自行檢視後手動執行 |
| [SKILL.md](SKILL.md) | 這次還原 ER 關聯所用方法的可重用 Skill 說明 |

## 這次分析遵守的規則

- 全程只對資料庫執行**唯讀 SELECT**（查詢 `INFORMATION_SCHEMA` / `sys.foreign_keys` 等 metadata，
  以及資料完整性檢查），沒有執行任何 DML（INSERT/UPDATE/DELETE）或 DCL（GRANT/REVOKE）。
- 產生的 `add_constraints.sql` 內含 DDL（`ALTER TABLE ADD CONSTRAINT`），
  但同樣**沒有**代為執行，只是產出腳本供你自行決定是否套用。

## 關聯是怎麼判斷出來的

沒有單純靠欄位名稱猜測。實際作法：

1. 直接讀取原始資料庫 `biz00` 裡「已經存在」的 PK / FK 定義（該資料庫設計者當初就有建好完整約束）。
2. 把這組關聯拿去對 `biz_1024`（複製後的資料）做完整性驗證：
   - 每個預定 PK 欄位有沒有重複值
   - 每個預定 FK 欄位有沒有對應不到父表的「孤兒值」
3. 驗證結果全部 0 筆違反，代表這組關聯定義套用到 `biz_1024` 上是合法、可以直接執行的。

詳細方法已抽象成 [SKILL.md](SKILL.md)，之後遇到類似「資料庫複製後遺失約束、需要重建 ER 圖」的
任務可以直接照著做。

---
> Source: [kevicky101-lab/1024imsocool](https://github.com/kevicky101-lab/1024imsocool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
