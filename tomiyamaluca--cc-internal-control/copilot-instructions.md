## cc-internal-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

このファイルは、drawio業務フロー図作成プロジェクトにおけるClaude Codeへのガイドラインです。

<!-- ============================================================
 CLAUDE.md  ── drawio業務フロー図作成プロジェクト
 Last updated: 2025-06-10
=============================================================== -->

## 🎯 プロジェクトの目的
- **成果物**: SOX法対応の業務フロー図（drawio形式）
- **対象**: 内部統制・監査部門向け業務プロセス文書
- **規格**: J-SOX（金融商品取引法）準拠の内部統制文書

---

## 📊 業務フロー図の基本構造

### スイムレーン構成例（左→右）必要に応じて加除する
1. **社外** - 顧客、仕入先、配送業者など
2. **店舗担当者** - 販売員、レジ担当、売場担当など
3. **店舗管理者** - 店長、副店長、売場主任など
4. **本社営業担当者** - 商品企画、仕入担当、販促担当など
5. **本社営業管理者** - 営業部長、商品部長、販促部長など
6. **本社管理部門担当者** - 経理、人事、総務などの実務担当者
7. **本社管理部門承認者** - 経理部長、人事部長、CFOなど
8. **社内システム** - POSシステム、在庫管理システム、ERPなど
9. **社外サービス** - 決済代行、配送業者API、外部サービスなど

### drawio設定
```xml
<!-- ページ設定 -->
<mxGraphModel grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1654" pageHeight="1169" math="0" shadow="0">
  <!-- A3横向き: 1654x1169px -->
</mxGraphModel>
```

---

## 🔍 SOX統制要素

### リスク分類
| 記号 | リスク種別 | 色 | 説明 |
|------|-----------|-----|------|
| **R1** | 財務リスク | #FFE6E6 | 財務報告の虚偽記載リスク |
| **R2** | 業務リスク | #FFF0E6 | 業務プロセスの不正・誤謬リスク |
| **R3** | コンプライアンスリスク | #E6F0FF | 法令違反リスク |
| **R4** | ITリスク | #F0E6FF | システム関連リスク |

### コントロール分類
| 記号 | コントロール種別 | 色 | 説明 |
|------|-----------------|-----|------|
| **C1** | 予防的統制 | #E6FFE6 | 事前にリスクを防止 |
| **C2** | 発見的統制 | #E6F5FF | 事後的にリスクを発見 |
| **KC** | キーコントロール | #FFD700 | 重要な統制（★マーク付） |

### IT統制分類
| 記号 | IT統制種別 | 色 | 説明 |
|------|-----------|-----|------|
| **IT-G** | IT全般統制 | #D3D3D3 | システム全体の統制 |
| **IT-A** | IT業務処理統制 | #B0E0E6 | 自動化された業務統制 |

---

## 📐 フロー図作成ルール

### 基本要素
1. **プロセス**: 角丸長方形（通常業務）
2. **判断**: ひし形（分岐・承認判断）
3. **文書**: 波型下辺の長方形（帳票・文書）
4. **データ**: 平行四辺形（データ入出力）
5. **システム処理**: 長方形（自動処理）

### 統制ポイントの表記（付箋形式）
```xml
<!-- プロセスボックスの例 -->
<mxCell id="process-001" value="販売処理&lt;br&gt;(P-001)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#DAEEFF;strokeColor=#000000;" parent="4" vertex="1">
  <mxGeometry x="25" y="140" width="120" height="60" as="geometry" />
</mxCell>

<!-- リスク付箋の例 -->
<mxCell id="risk-001" value="R-001&lt;br&gt;金額誤り" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#FFE6E6;strokeColor=#B85450;size=15;fontSize=10;" parent="4" vertex="1">
  <mxGeometry x="150" y="130" width="60" height="40" as="geometry" />
</mxCell>

<!-- 通常統制付箋の例 -->
<mxCell id="control-001" value="C-001&lt;br&gt;金額確認" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#E6FFE6;strokeColor=#82B366;size=15;fontSize=10;" parent="4" vertex="1">
  <mxGeometry x="150" y="175" width="60" height="40" as="geometry" />
</mxCell>

<!-- キーコントロール付箋の例 -->
<mxCell id="control-002" value="C-002 ★&lt;br&gt;現金実査" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#FFD700;strokeColor=#C8AB37;size=15;fontSize=10;fontStyle=1;" parent="4" vertex="1">
  <mxGeometry x="-40" y="250" width="60" height="40" as="geometry" />
</mxCell>

<!-- IT統制付箋の例 -->
<mxCell id="control-008" value="C-008&lt;br&gt;自動集計&lt;br&gt;[IT統制]" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#B0E0E6;strokeColor=#5D7F99;size=15;fontSize=10;" parent="6" vertex="1">
  <mxGeometry x="-40" y="560" width="60" height="40" as="geometry" />
</mxCell>
```

### 接続線のルール
- **実線**: 通常のフロー
- **破線**: 情報の参照・照会
- **太線**: キーコントロールを含むフロー
- **赤線**: リスクの高いフロー

---

## 📝 レジェンド構成

### 必須レジェンド項目
```xml
<mxCell value="【凡例】" style="text;html=1;fontSize=14;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="20" y="20" width="100" height="30" as="geometry"/>
</mxCell>

<!-- プロセス記号 -->
<mxCell value="通常プロセス" style="shape=process;fillColor=#E6F3FF;" vertex="1" parent="1">
  <mxGeometry x="20" y="60" width="100" height="40" as="geometry"/>
</mxCell>

<!-- リスク記号 -->
<mxCell value="R1: 財務リスク" style="shape=hexagon;fillColor=#FFE6E6;" vertex="1" parent="1">
  <mxGeometry x="20" y="110" width="100" height="40" as="geometry"/>
</mxCell>

<!-- コントロール記号 -->
<mxCell value="C1: 予防的統制" style="shape=parallelogram;fillColor=#E6FFE6;" vertex="1" parent="1">
  <mxGeometry x="20" y="160" width="100" height="40" as="geometry"/>
</mxCell>

<!-- キーコントロール -->
<mxCell value="KC: キーコントロール ★" style="shape=process;fillColor=#FFD700;strokeWidth=2;" vertex="1" parent="1">
  <mxGeometry x="20" y="210" width="150" height="40" as="geometry"/>
</mxCell>

<!-- IT統制 -->
<mxCell value="IT-A: 自動統制" style="shape=rectangle;fillColor=#B0E0E6;" vertex="1" parent="1">
  <mxGeometry x="20" y="260" width="100" height="40" as="geometry"/>
</mxCell>
```

---

## 🎨 配色ガイドライン

### スイムレーン背景色
- **社外**: #F5F5F5（薄いグレー）
- **フロント**: #E6F3FF（薄い青）
- **ミドルオフィス**: #F0F8E6（薄い緑）
- **バックオフィス**: #FFF0E6（薄いオレンジ）
- **システム**: #F0E6FF（薄い紫）

### プロセス要素の配色
- **通常プロセス**: 各スイムレーンの背景色を少し濃くした色
  - フロント系: #DAEEFF
  - ミドル系: #E6F0DC
  - バック系: #FFE8D8
  - システム系: #E8D8FF
- **承認・判断**: #FFE6E6（薄い赤）

### 付箋（リスク・コントロール）の配色
- **リスク**: #FFE6E6（薄い赤）、枠線 #B85450
- **通常統制**: #E6FFE6（薄い緑）、枠線 #82B366
- **キーコントロール**: #FFD700（金色）、枠線 #C8AB37
- **IT統制**: #B0E0E6（薄い青）、枠線 #5D7F99

### テキストルール
- **フォント**: メイリオ または ヒラギノ角ゴ
- **サイズ**: 
  - タイトル: 18pt
  - スイムレーンヘッダー: 12pt（改行を使用して2行表示）
  - プロセス名: 11pt
  - 付箋内テキスト: 10pt

---

## 📋 成果物チェックリスト

### 必須要素
- [ ] 全スイムレーンが定義されている
- [ ] 開始と終了が明確
- [ ] すべてのリスクポイントが識別されている
- [ ] 各リスクに対応するコントロールが配置されている
- [ ] キーコントロールが★マークで識別されている
- [ ] IT統制ポイントが明記されている
- [ ] レジェンドが完備されている
- [ ] 承認フローが明確
- [ ] 文書・帳票の流れが追跡可能

### 品質基準
- [ ] フローの流れが論理的で追跡可能
- [ ] 統制の有効性が視覚的に確認できる
- [ ] J-SOX文書化要件を満たしている
- [ ] 監査証跡の取得ポイントが明確

---

## 🚀 作業手順（テンプレート必須使用）

### 0. 事前準備
- **必須**: `テンプレート`フォルダ内のテンプレートファイルを使用
  - `業務記述書_テンプレート.md`
  - `RCM_テンプレート.md` 
  - `業務フロー図_テンプレート.drawio`
- **重要**: テンプレートを必ずコピーしてから編集開始（直接編集禁止）
- プロセス番号の採番ルール確認（P-001〜、R-001〜、C-001〜、S-001〜）

### 1. 業務分析とプロセス定義
- プロセスの開始点と終了点を明確化
- 主要なプロセスステップを洗い出し（通常10〜15ステップ）
- 各ステップの実施者（部門・役職）を特定
- 承認ポイントを識別

### 2. リスクと統制の識別
- 各プロセスステップのリスクを識別
  - 財務リスク（虚偽記載、計上誤り等）
  - 業務リスク（紛失、横領、処理漏れ等）
  - コンプライアンスリスク（法令違反等）
- 各リスクに対する統制活動を定義
- キーコントロール（最重要統制）を選定（通常3〜4個）

### 3. 業務記述書の作成（テンプレート使用）
```bash
# テンプレートをコピー
cp "業務記述書_テンプレート.md" "[プロセス名]_業務記述書_v1.0_YYYYMMDD.md"
```
- テンプレート構造を維持してカスタマイズ
- プロセス概要（目的、範囲、関連部門）
- プロセス詳細：大きなテーブル形式で以下を記載
  - プロセス番号・名称・実施者
  - 活動内容の詳細
  - 使用証憑（インプット）：具体的な帳票名・文書名
  - 作成証憑（アウトプット）：作成される成果物
  - 関連勘定科目：該当する勘定科目（補助科目含む）
  - リスク：識別されたリスク（ない場合は「-」）
  - 統制：対応する統制活動（ない場合は「-」、キーコントロールは★付き）
  - システム連携：関連するシステム処理
- リスク（R-001〜R-016）と統制（C-001〜C-013）の一覧表
- システム連携情報（S-001〜S-003）
- 監査証跡の詳細リスト（種別・保存期間・保存場所含む）

### 4. RCM（Risk Control Matrix）の作成（テンプレート使用）
```bash
# テンプレートをコピー
cp "RCM_テンプレート.md" "[プロセス名]_RCM_v1.0_YYYYMMDD.md"
```
- **重要**: 7種類のアサーション列を必須記載
  - **実在性（Existence）**: 取引が実際に発生したか
  - **網羅性（Completeness）**: 全ての取引が記録されているか
  - **正確性（Accuracy）**: 金額・日付が正確か（※RCMでは権利義務列として記載）
  - **期間帰属（Cut-off）**: 適切な期間に計上されているか
  - **表示開示（Presentation & Disclosure）**: 適切に表示・開示されているか
  - **評価測定（Valuation & Allocation）**: 適切に評価・測定されているか
  - **権利義務（Rights & Obligations）**: 権利・義務が適切に反映されているか
- **重要**: 不正リスク列を必須記載（表示開示の右側）
  - **不正リスク（Fraud Risk）**: 意図的な虚偽表示・資産流用・収賄等の不正に該当する場合「X」マーク
- **必須**: リスク評価3項目をHML表記
  - **発生可能性**: H（高）/ M（中）/ L（低）
  - **影響度**: H（高）/ M（中）/ L（低）
  - **リスクレベル**: H×H=最高, H×M,M×H=高, M×M,H×L,L×H=中, M×L,L×M,L×L=低
- 大きなMDテーブル形式でリスクと統制を一覧表示
- キーコントロールを明示：
  - 業務記述書・フロー図：★マーク使用
  - RCM：KC列に「X」マーク使用
- IT統制関連情報（IT依存度、関連IT統制）を記載

### 5. 業務記述書とRCMの整合性確認
- プロセス番号（P-001〜P-013）の一致確認
- リスク番号（R-001〜R-016）の一致確認
- 統制番号（C-001〜C-013）の一致確認
- キーコントロールの統一確認（業務記述書・フロー図の★とRCMのKC列「X」）
- 不整合箇所の修正

### 6. 業務フロー図の作成（テンプレート使用）
```bash
# テンプレートをコピー
cp "業務フロー図_テンプレート.drawio" "[プロセス名]_業務フロー図_v1.0_YYYYMMDD.drawio"
```
- **重要**: 開始ノード（緑色楕円）と終了ノード（赤色楕円）を必須配置
- スイムレーンの調整（実際の部門名に変更）
- プロセスボックスの配置（上から下へ時系列）
- リスク・統制の付箋を適切な位置に配置
  - リスクは右側に配置（#FFE6E6）
  - 統制は左側または上側に配置（#E6FFE6）
  - キーコントロールは金色（#FFD700）で★マーク付
  - IT統制は水色（#B0E0E6）
- フロー線の接続（実線：通常フロー、破線：システム連携）
- レジェンドの完備

### 7. 相互参照の確認
- プロセス番号（P-XXX）の3文書間完全一致確認
- リスク番号（R-XXX）の3文書間完全一致確認
- 統制番号（C-XXX）の3文書間完全一致確認
- システム連携番号（S-XXX）の3文書間完全一致確認
- キーコントロールの3文書間統一確認（業務記述書・フロー図の★とRCMのKC列「X」）

### 8. 品質チェック
- 全てのリスクに対応する統制があるか
- キーコントロールが適切に設定されているか（3〜4個）
  - 業務記述書・フロー図：★マーク
  - RCM：KC列に「X」
- 承認フローが明確か（ひし形で表現）
- 開始・終了ノードが配置されているか
- 7種類のアサーションが全て記載されているか
- **リスク評価3項目がHML表記されているか**
  - 発生可能性：H/M/L
  - 影響度：H/M/L  
  - リスクレベル：H/M/L
- 監査証跡が十分か
- 3文書（業務記述書、RCM、フロー図）の完全な内容整合性

---

## 📄 ファイル命名規則
```
[プロセス名]_業務フロー図_[バージョン]_[日付].drawio
例: 購買プロセス_業務フロー図_v1.0_20250108.drawio
```

---

## ⚠️ 注意事項
- 個人情報や機微情報は記載しない
- 実在の取引先名は「取引先A」などに置換
- システム名は一般名称を使用
- 金額は「XXX円」として具体額は記載しない(金額基準の承認権限などの例外あり)

---

## 🔧 技術仕様

### drawio XML構造
```xml
<mxfile host="app.diagrams.net" modified="2025-01-08" agent="5.0">
  <diagram name="業務フロー図" id="unique-id">
    <mxGraphModel dx="1422" dy="794" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1654" pageHeight="1169">
      <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- スイムレーン定義 -->
        <!-- プロセス要素 -->
        <!-- 接続線 -->
        <!-- レジェンド -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

### 推奨エクスポート形式
- **編集用**: .drawio（XML形式）
- **配布用**: PDF（A3横、ベクター形式）
- **プレビュー用**: PNG（高解像度）

<!-- ============ End of CLAUDE.md =================================== -->

## 🛠️ Common Commands and Development Workflow

### ファイル操作コマンド
```bash
# テンプレートをコピーして新規プロジェクト開始
cp "テンプレート/業務記述書_テンプレート.md" "[プロセス名]_業務記述書_v1.0_YYYYMMDD.md"
cp "テンプレート/RCM_テンプレート.md" "[プロセス名]_RCM_v1.0_YYYYMMDD.md"
cp "テンプレート/業務フロー図_テンプレート.drawio" "[プロセス名]_業務フロー図_v1.0_YYYYMMDD.drawio"

# drawioファイルのXML構造確認
xmllint --format "[プロセス名]_業務フロー図_v1.0_YYYYMMDD.drawio"
```

### 文書間整合性チェック
- プロセス番号（P-XXX）の確認: `grep -E "P-[0-9]{3}" *.md *.drawio`
- リスク番号（R-XXX）の確認: `grep -E "R-[0-9]{3}" *.md *.drawio`
- 統制番号（C-XXX）の確認: `grep -E "C-[0-9]{3}" *.md *.drawio`
- キーコントロールの確認: 
  - 業務記述書・フロー図: `grep "★" *.md *.drawio`
  - RCM: KC列の「X」を目視確認

## 📁 High-Level Architecture

### ドキュメント連携構造
```
業務記述書.md ←→ RCM.md ←→ 業務フロー図.drawio
     ↓             ↓              ↓
  プロセス定義   リスク評価    視覚的表現
  (P-001〜)     (7アサーション) (スイムレーン)
     ↓             ↓              ↓
  統制活動      統制有効性     統制配置
  (C-001〜)     (HML評価)      (付箋形式)
```

### テンプレート依存関係
- **業務記述書_テンプレート.md**: 基本プロセス定義（必須開始点）
- **RCM_テンプレート.md**: リスク・統制マトリクス（業務記述書に基づく）
- **業務フロー図_テンプレート.drawio**: 視覚化（業務記述書とRCMに基づく）

### 重要な整合性ポイント
1. **番号体系の統一**: P-XXX, R-XXX, C-XXX, S-XXXは3文書で完全一致必須
2. **キーコントロール**: 
   - 業務記述書・フロー図：★マーク
   - RCM：KC列に「X」
   - 通常3〜4個
3. **色コード**: 特にリスク（#FFE6E6）と統制（#E6FFE6, #FFD700, #B0E0E6）
4. **アサーション**: RCMの7列は省略不可

## 📌 Critical Rules for Claude Code

1. **テンプレート使用は絶対**: 新規作成時は必ずテンプレートフォルダからコピー
2. **3文書セット**: 業務記述書→RCM→フロー図の順で作成
3. **番号採番**: 連番で管理（P-001, P-002...）、欠番は作らない
4. **drawio編集時**: XMLタグのid属性は意味のある名前を使用（例: id="process-001", id="risk-001"）
5. **リスク評価**: 必ずH/M/L形式で記載（High/Medium/Low）

---
> Source: [tomiyamaluca/CC_Internal_Control](https://github.com/tomiyamaluca/CC_Internal_Control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
