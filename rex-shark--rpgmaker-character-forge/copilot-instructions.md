## rpgmaker-character-forge

> 本專案以 RPG Maker MV／MZ 的單人行走圖、face 圖或完成立繪作為角色設計

# RPG Maker Character Forge：Agent 與 Job 使用指南

## 專案目的

本專案以 RPG Maker MV／MZ 的單人行走圖、face 圖或完成立繪作為角色設計
參考，透過 Codex Agent、繪圖工具、Python 與 ComfyUI 生成：

- 精緻全身立繪
- RPG Maker 4×2 face 圖
- `sv_enemies` 敵人戰鬥圖

目前只保留正式的黑底流程 `job1`～`job6`，生圖背景固定為純黑
`#000000`。

## Job 一覽

| Job | 使用者輸入 | 產物 |
| --- | --- | --- |
| Job 1 | 一張單人行走圖 | 透明全身立繪 |
| Job 2 | 單人行走圖＋face 圖 | 透明全身立繪 |
| Job 3 | 完成立繪 | 16 表情、兩張透明 4×2 face 圖 |
| Job 4 | 完成立繪＋單人行走圖 | 透明 `sv_enemies` 戰鬥圖 |
| Job 5 | 一張單人行走圖 | 依序完成 Job 1、3、4 的四張成品 |
| Job 6 | 單人行走圖＋face 圖 | 透明立繪＋透明戰鬥圖 |

Job 定義與使用者資料已分離。對應關係固定如下：

| Job | 定義目錄 | 工作目錄 |
| --- | --- | --- |
| Job 1 | `jobs/01-sprite-to-illustration/` | `workspace/job01/` |
| Job 2 | `jobs/02-sprite-face-to-illustration/` | `workspace/job02/` |
| Job 3 | `jobs/03-illustration-to-face-sheet/` | `workspace/job03/` |
| Job 4 | `jobs/04-illustration-sprite-to-sv-enemy/` | `workspace/job04/` |
| Job 5 | `jobs/05-sprite-to-complete-set/` | `workspace/job05/` |
| Job 6 | `jobs/06-sprite-face-to-battle-set/` | `workspace/job06/` |

## Agent 共通規則

執行 Job 前必須完整讀取該目錄的 `AGENTS.md`、`system_prompt.txt`、
`prompt.schema.json` 與相關輸出 Schema；Job 專用規則優先於本文件。

所有目前的 Job 都採黑底流程。Job 1、Job 5 的 Job 1 階段與 Job 6 在圖片
分析前多一個 ComfyUI 4× 行走圖準備步驟；其餘 Job 直接從既有參考圖開始：

1. 依 Job 規則準備參考圖；Job 1 使用 ComfyUI 4× 放大。
2. Agent 分析實際要交給繪圖工具的參考圖。
3. 組合提示詞並保存 `composed_prompt.json`。
4. 繪圖工具在純黑 `#000000` 背景生圖。
5. ComfyUI 完成去背、縮放及必要的裁切／分割。
6. Agent 比對原始參考、實際生圖參考、黑底原圖與透明輸出，保存 AI 視覺驗證。

只有 Job 1、Job 5 的 Job 1 階段與 Job 6 放大行走圖；Job 2、4 依各自規則
使用原圖。所有流程都不使用 Pillow 背景校正、舊共用自動驗證器或棋盤
預覽。不得因 ComfyUI JSON 內的預設 `image` 名稱要求使用者重新命名；執行器
會動態替換唯一的 `LoadImage` 與終端 `SaveImage`。

未經使用者同意不得覆寫既有 run；需要覆寫時才可使用 `--force`。不得擅自
把成品複製到其他 RPG Maker 專案。

每個 Job 通過 AI 驗證後，必須把本次實際使用的所有輸入參考圖，以
`<角色名稱>-reference-*.png` 固定名稱複製到同一 run 的 `output/`。參考圖
副本必須與原始輸入位元組完全相同，不得縮放、裁切、重新編碼或以生成圖
替代。未經同意不得覆寫既有參考圖副本。

## 全身生圖共同規格

Job 1、Job 2、Job 4、Job 5 對應階段，以及 Job 6 的兩個階段必須遵守：

- 黑底原圖畫布為寬 432、高 576 像素。
- 背景均勻純黑，精確色碼 `#000000`；不得有場景、地面、陰影、文字、框線
  或漸層。
- 人物採非 Q 版的標準動漫／JRPG 全身比例，約 6.5～7 頭身；頭部約占
  全身七分之一，四肢自然修長，腿部約占全身一半。
- 行走圖與 face 近拍只提供設計與身份，不能控制身體比例。禁止超大頭、
  幼兒或玩偶身材、五頭身以下及短小四肢。
- 原角色的髮型、服裝、配色、裝備、配件與身份特徵必須盡可能保留；像素圖
  無法確認的細節不得當成事實。

每一份全身生圖提示詞必須逐字以下列文字開頭，使用者提示詞不可覆蓋：

```text
Create exactly one single character in exactly one full-body view. This is one standalone character illustration, not a turnaround, model sheet, reference sheet, comparison, triptych, or multi-view layout. Never show duplicate bodies, side views, back views, panels, or multiple poses. Use side and rear sprite frames only to understand hidden design details; do not display those additional views in the output.
```

行走圖內的正面、側面與背面畫格只能用來推斷同一人物的設計，絕對不得生成
三視圖、角色設定表、轉面圖、分格、並排人物、重複身體或多個姿勢。

只有使用者未指定替代方案時，才加入 JRPG、全身、賽璐璐與奇幻異世界等
條件式預設。若使用者指定寫實油畫等替代風格，不再加入衝突預設；但黑底、
尺寸、單人物單視角、身份一致與 Job 專用參考圖責任永遠不可覆蓋。

## Agent AI 視覺驗證標準

AI 驗證優先判斷成品是否實際可用：

1. 是否仍可辨識為參考圖中的同一人物。
2. 是否符合單人物、單身體、單姿勢與單一可見視角。
3. 人物、臉部表情格、肢體、服裝、鞋履、武器或主要配件有無重大缺失、
   錯誤裁切或破損。
4. ComfyUI 去背、縮放、裁切或分割後，有無明顯吃掉臉、頭髮、四肢、服裝、
   武器或其他重要細節。
5. 背景是否在視覺上有效移除，透明 PNG 是否可正常使用。

下列狀況本身不得造成失敗：黑底接近而非逐像素等於 `(0,0,0)`、透明區
Alpha 略高於 0、Alpha 1～31 的低殘值、一像素細邊、輕微光邊、少量留白
或細微比例差異。只要一般尺寸及亮／暗底預覽沒有可見色塊、方框、明顯光暈
或人物破損即可通過，並把輕微瑕疵寫入 `notes`。

## Job 1：行走圖生成立繪

輸入：

```text
workspace/job01/input/
├── <角色名稱>.png
└── prompt.json（可省略）
```

```json
{
  "檔名": "艾莉絲.png",
  "user_prompt": "手持銀色長劍，神情自信"
}
```

執行：`執行 job1`

原始行走圖先以 `tools/comfyui/workflows/upscale_sprite_4x.json` 使用
`4x-UltraSharpV2.pth` 放大 4×。Agent 分析該放大圖，繪圖工具也只接收該
放大圖，不再同時傳入原圖。黑底原圖再直接交給
`tools/comfyui/workflows/remove_background_and_fit.json`，輸出固定 432×576 的透明 PNG。

```text
workspace/job01/runs/<角色名稱>/
├── composed_prompt.json
├── prepared/<角色名稱>-sprite-4x.png
├── generated/<角色名稱>-black-source.png
├── output/
│   ├── <角色名稱>-reference-sprite.png
│   ├── <角色名稱>-reference-sprite-4x.png
│   └── <角色名稱>-transparent-trimmed.png
└── ai_validation.json
```

## Job 2：行走圖＋face 圖生成立繪

輸入：

```text
workspace/job02/input/
├── characters/<角色行走圖>.png
├── faces/<角色face圖>.png
└── prompt.json（可省略）
```

```json
{
  "characters": "艾莉絲.png",
  "face": "艾莉絲.png",
  "user_prompt": "微笑，單手叉腰"
}
```

執行：`執行 job2`

繪圖工具依序接收「原始行走圖＋原始 face 圖」。face 圖優先控制臉型、
五官、髮際線與臉部身份；行走圖優先控制身體、服裝、配色、裝備及背面設計。
face 近拍不得放大成品頭部。

```text
workspace/job02/runs/<角色名稱>/
├── composed_prompt.json
├── generated/<角色名稱>-black-source.png
├── output/
│   ├── <角色名稱>-reference-sprite.png
│   ├── <角色名稱>-reference-face.png
│   └── <角色名稱>-transparent-trimmed.png
└── ai_validation.json
```

## Job 3：立繪生成 16 表情 face 圖

可以接續已通過 AI 驗證的 Job 1 run，或直接提供一張完成立繪：

```text
workspace/job03/input/illustrations/<角色名稱>.png
```

`prompt.json` 依 `prompt.schema.json` 使用 `job1` 或 `direct` 模式。執行：
`執行 job3`，或指定接續的角色 run。

繪圖工具只接收完成立繪，生成純黑背景 576×576、4 欄×4 列的同一角色
16 表情。每格相當於 144×144，人物固定為朝畫面左側約 15～20° 的頭肩
近拍；完成立繪只控制角色身份與造型，不控制原始頭部、視線或肩膀方向。
即使立繪面向右側，也必須將同一角色重新繪製為由正面朝畫面左側旋轉約
30°；鼻尖與下巴朝畫布左緣，兩眼保持可見。

固定表情順序：

```text
0 中立      1 微笑／開心  2 生氣      3 受傷／痛苦
4 驚訝      5 悲傷／擔心  6 堅定／嚴肅 7 戰敗／昏厥
8 害羞      9 鬼臉       10 大哭      11 水汪汪大眼
12 挑逗    13 睡覺       14 困惑／疑問 15 嘟嘴
```

黑底原圖直接交給 `tools/comfyui/workflows/remove_background_and_split_face_sheet.json`，由 ComfyUI 去背、重組並上下切成
兩張 576×288、4×2 的透明 face 圖。

```text
workspace/job03/runs/<角色名稱>/
├── composed_prompt.json
├── generated/<角色名稱>-expressions-16-black-source.png
├── output/
│   ├── <角色名稱>-reference-illustration.png
│   ├── <角色名稱>-faces-1.png
│   └── <角色名稱>-faces-2.png
└── ai_validation.json
```

## Job 4：敵人戰鬥圖（sv_enemies）

有兩種輸入模式：

- `job1`：接續通過 AI 驗證的 Job 1 run。
- `direct`：指定原始單人行走圖與完成立繪。

詳細 JSON 見 `jobs/04-illustration-sprite-to-sv-enemy/prompt.job1.example.json`、
`prompt.direct.example.json` 與 `prompt.schema.json`。執行：`執行 job4`。

繪圖工具依序接收「完成立繪＋原始行走圖」。完成立繪是臉部、正面服裝、
腿部服飾、鞋履、比例與畫風的第一主參考；行走圖只補充看不到的背面及側面
資訊，不得覆蓋立繪可見設計。

人物固定面向畫面右側，預設只要求一般戰鬥姿勢，不額外指定低重心或防守
架勢。使用者未指定表情時使用嚴肅、認真、緊張且警戒的戰鬥表情。禁止
魔法、粒子、光芒、煙霧、斬擊軌跡與其他戰鬥特效；武器依使用者提示、
參考圖或明確職業保守判定，資訊不足時不新增。

```text
workspace/job04/runs/<角色名稱>/
├── composed_prompt.json
├── generated/<角色名稱>-sv-enemy-black-source.png
├── output/
│   ├── <角色名稱>-reference-illustration.png
│   ├── <角色名稱>-reference-sprite.png
│   └── <角色名稱>-sv-enemy-transparent-trimmed.png
└── ai_validation.json
```

## Job 5：單張行走圖統一生成完整素材

輸入：

```text
workspace/job05/input/
├── <角色名稱>.png
└── prompt.json
```

```json
{
  "檔名": "艾莉絲.png",
  "illustration_prompt": "立繪需求",
  "face_prompt": "face 圖需求",
  "battle_prompt": "戰鬥圖需求"
}
```

執行：`執行 job5`

Job 5 固定依序執行 Job 1 → Job 3 → Job 4。這是一次使用者指令、三次分開
生圖；上一階段 `ai_validation.json` 未通過不得繼續。三組提示詞只傳給各自
階段，不得交叉混用。最後彙整透明立繪、兩張 face 圖與透明戰鬥圖。

```text
workspace/job05/runs/<角色名稱>/
├── composed_prompt.json
├── stage_inputs/
├── stage_validations/
└── output/
    ├── <角色名稱>-reference-sprite.png
    ├── <角色名稱>-illustration-transparent.png
    ├── <角色名稱>-faces-1.png
    ├── <角色名稱>-faces-2.png
    └── <角色名稱>-sv-enemy-transparent.png
```

## Job 6：行走圖＋face 圖生成立繪與戰鬥圖

輸入：

```text
workspace/job06/input/
├── characters/<角色行走圖>.png
├── faces/<角色face圖>.png
└── prompt.json
```

```json
{
  "characters": "艾莉絲.png",
  "face": "艾莉絲.png",
  "illustration_prompt": "立繪需求",
  "battle_prompt": "戰鬥圖需求"
}
```

執行：`執行 job6`

原始行走圖先以 `tools/comfyui/workflows/upscale_sprite_4x.json` 放大 4×，且
全流程只放大一次。立繪生圖固定接收「4× 放大行走圖＋原始 face 圖」。立繪
通過 AI 驗證後，戰鬥圖生圖固定接收「同一張 4× 放大行走圖＋同一張原始
face 圖＋本次透明立繪」。完成放大後，不再把原始行走圖傳給繪圖工具。
兩組提示詞隔離，face 圖優先控制臉部身份，放大行走圖優先控制服裝與身體，
完成立繪補充已建立的細節、比例及畫風連貫性。

```text
workspace/job06/runs/<角色名稱>/
├── composed_prompt.json
├── prepared/<角色名稱>-sprite-4x.png
├── prompt_records/
├── generated/
├── output/
│   ├── <角色名稱>-reference-sprite.png
│   ├── <角色名稱>-reference-sprite-4x.png
│   ├── <角色名稱>-reference-face.png
│   ├── <角色名稱>-illustration-transparent-trimmed.png
│   └── <角色名稱>-sv-enemy-transparent-trimmed.png
└── stage_validations/
```

## 靜態前端發布

前端位於 `frontend/frontend/`，只讀取
`frontend/frontend/public/characters/index.json` 與各角色的
`character.json`，不直接依賴 `workspace/`，也不連接後端。

Job 5 或 Job 6 的所有必要階段通過 AI 驗證後，使用共用發布工具複製素材並
更新前端索引：

```bash
python3 tools/gallery/publish_character.py workspace/job05/runs/<角色名稱>
python3 tools/gallery/publish_character.py workspace/job06/runs/<角色名稱>
```

發布副本不得取代或修改原始 run。既有展示圖片內容不同時，工具預設拒絕
覆寫；只有使用者確認更新後才可使用 `--force`。批次與自訂名稱說明見
`tools/gallery/README.md`。

## ComfyUI 工具

- `tools/comfyui/workflows/upscale_sprite_4x.json`：Job 1、Job 5 的 Job 1 階段與 Job 6 使用
  `4x-UltraSharpV2.pth` 將原始單人行走圖放大 4×，供 Agent 分析及繪圖工具參考。
- `tools/comfyui/workflows/remove_background_and_fit.json`：Job 1、2、4、6 的全身圖黑底去背、
  置中縮放與固定 432×576 輸出。
- `tools/comfyui/workflows/remove_background_and_split_face_sheet.json`：Job 3 的 4×4 黑底 face 原圖去背、重組並切成兩張
  576×288 圖片。
- `tools/comfyui/run_workflow.py`：所有 Job 共用的執行器，負責動態上傳輸入、替換工作流節點、排入
  ComfyUI、等待並下載終端輸出。

本機 ComfyUI 預設網址為 `http://127.0.0.1:8188`。可先執行：

```bash
curl -sS http://127.0.0.1:8188/system_stats
```

有 JSON 回應即代表目前執行環境可連線；瀏覽器能開不保證另一個隔離 session
也能存取主機服務。

## 規格測試

```bash
python3 -m unittest discover -s tests/tools -p 'test_*.py' -v
python3 -m unittest discover -s tests/jobs -p 'test_*.py' -v
```

---
> Source: [Rex-shark/rpgmaker-character-forge](https://github.com/Rex-shark/rpgmaker-character-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
