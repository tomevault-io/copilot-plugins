## reelmimic

> 這是「風格克隆影片工作室」：使用者給參考影片＋需求 → 拆解風格 → 選製作技能 → 前製企劃（與使用者討論到核准）→ 生成 → 依回饋修改。

# AGENTS.md（Codex 讀這份；Claude Code 讀 CLAUDE.md，兩份規則相同）

這是「風格克隆影片工作室」：使用者給參考影片＋需求 → 拆解風格 → 選製作技能 → 前製企劃（與使用者討論到核准）→ 生成 → 依回饋修改。

- 流程與規則：`.claude/skills/video-clone/SKILL.md`
- 檔案格式（必須照寫）：`.claude/skills/video-clone/CONTRACT.md`
- 風格對應表：`.claude/skills/video-clone/styles/*.md`
- 各製作技能的說明：`.claude/skills/<engine>/SKILL.md`（例如 painted-animation、hyperframes）。Codex 沒有 Skill 工具，直接讀這些檔案照做。
- 分析：`python .claude/skills/video-clone/scripts/analyze.py <檔案或網址> --out <專案>/analysis`
- 授權安全素材：`python .claude/skills/video-clone/scripts/fetch_assets.py search|get …`

規則：目前只做 2D（3D 暫停），3D 參考片用 2D 重現並明講；不生成真人實拍或真實人物；使用者自己擁有的角色（提供設計圖）可直接用，其他角色與畫面原創也可直接使用；歌詞只用使用者提供的檔案，不自行寫出；參考片只學手法不抄素材；
每次渲染後打開影格檢查，修到好才輸出。FFmpeg 找不到時看環境變數 `FFMPEG_DIR`。

---
> Source: [edenfunf/reelmimic](https://github.com/edenfunf/reelmimic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
