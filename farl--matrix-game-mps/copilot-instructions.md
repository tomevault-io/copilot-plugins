## matrix-game-mps

> - **絕對禁止使用 `rm -rf` 刪除任何目錄或文件**

# CLAUDE.md - AI 記憶文檔

## 🚨 每次對話必讀的核心要點

### 安全操作原則
- **絕對禁止使用 `rm -rf` 刪除任何目錄或文件**
- **每個操作前先檢查文件是否存在，保護現有內容**
- **使用 `cp` 複製而不是移動，保留備份**
- **每完成一個步驟立即用 TodoWrite 標記完成**
- **出現任何錯誤立即停止，不要繼續破壞**

### 項目狀態
- **目標**：將 https://huggingface.co/Skywork/Matrix-Game-2.0 適配到 Apple Silicon (M2 Mac)
- **現狀**：大部分工作已丟失，只剩下 `README_Apple_Silicon.md` 技術文檔
- **任務**：根據 README_Apple_Silicon.md 記錄，從零重建整個適配版本

### 現存的重要文件（必須保護）
- ✅ `README_Apple_Silicon.md` (438行) - 唯一救回的寶貴技術文檔
- ✅ `temp_restore/Matrix-Game-2.0/` - 部分 Hugging Face 模型文件

### 關鍵技術要點（來自 README）
- **M2 Mac 限制**: MPS 最大 ~18GB，基本推理可行，串流推理記憶體不足
- **核心適配需求**: Flash Attention 回退、MPS 數據類型修正、NVIDIA 依賴移除
- **重要設定**: `PYTORCH_MPS_HIGH_WATERMARK_RATIO=0.0`

### 詳細計劃位置
完整執行計劃請查看：`PROJECT_PLAN.md`

### 執行檢查清單
每個步驟執行前檢查：
- [ ] 是否會刪除現有文件？（如果是，停止！）
- [ ] 是否有備份保護機制？
- [ ] TodoWrite 是否已更新？
- [ ] 文件路徑是否正確？
- [ ] 命令是否安全？

## 重要提醒
**這是無狀態AI的記憶文檔，每次操作前必須先讀取 CLAUDE.md 和 PROJECT_PLAN.md！**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Farl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
