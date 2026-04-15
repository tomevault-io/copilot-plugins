## holodelive

> - `docs/game-rules.md` - ゲームルール詳細

# HOLOdeLIVE

2人専用ターン制カードゲーム。

## プロジェクト構成

- `docs/game-rules.md` - ゲームルール詳細
- `docs/game-architecture.md` - データ構造・アーキテクチャ設計
- `docs/session-and-multiplayer.md` - GameSession & マルチプレイ設計
- `test/` - ユニットテスト（GdUnit4）
- `addons/gdUnit4/` - テストフレームワーク

## テスト

- フレームワーク: [GdUnit4](https://github.com/godot-gdunit-labs/gdUnit4) v6.1.1
- テストファイルは `test/` 以下に、テスト対象と対応するディレクトリ構造で配置する
  - 例: `system/game_state.gd` → `test/system/game_state_test.gd`
- テストクラスは `GdUnitTestSuite` を継承し、テストメソッドは `test_` プレフィックスを付ける
- CLI実行: `GODOT_BIN=/usr/local/bin/godot ./addons/gdUnit4/runtest.sh -a test/`
- 全テスト実行時は `... 2>&1 | grep FAILED` で失敗だけ抽出する（出力が空なら全パス）
- 失敗があればそのファイルだけ再実行して詳細を確認: `GODOT_BIN=/usr/local/bin/godot ./addons/gdUnit4/runtest.sh -a test/path/to/failing_test.gd`
- XML/HTML レポートはトークン消費が大きいので読まない
- 新しい `class_name` を持つファイルを追加・削除した後は、テスト前にクラスキャッシュの再構築が必要:
  - `godot --headless --editor --quit`
  - `--headless --quit` だけでは `global_script_class_cache.cfg` が更新されないので注意
- 新規コード追加時は対応するテストも作成すること

## デバッグシーン（CLI テスト）

メインシーン (`scenes/debug/debug_scene.tscn`) はコマンドライン引数でゲーム状態と行動を制御できる。

### 実行方法

```bash
godot res://scenes/debug/debug_scene.tscn -- [ゾーンオーバーライド...] [auto=行動キュー] [cpu=both] [max_turns=N]
```

Makefile 経由:
```bash
make debug                              # 通常起動
make debug test=preset_id               # プリセット指定
make debug ARGS="p0=3,7 s1=47"          # 引数直指定
make debug test=9 cpu=both max_turns=30  # CPU対戦 + ターン制限
```

### ゾーンオーバーライド

ゲーム開始時のカード配置を指定する。値は card_id のカンマ区切り。`r` でランダム、末尾 `g` でゲスト（伏せ）配置。

| キー | ゾーン |
|------|--------|
| `p0/p1` | 手札 |
| `s0/s1` | ステージ |
| `b0/b1` | バックステージ |
| `h` | 自宅（共有） |
| `d` | デッキ先頭（指定順にドローされる） |

```bash
godot res://scenes/debug/debug_scene.tscn -- d=3,7 p0=45,10 s1=47 b1=2g h=20
```

プリセット: `test=preset_id`（`scenes/debug/test_presets.json` に定義）

### 自動行動キュー (`auto=`)

P0 の行動を事前予約する。P1 は CPU が自動応答。

```bash
godot res://scenes/debug/debug_scene.tscn -- p0=3,7 s1=47 auto=play:3:stage,select:7:stage,pass
```

コマンド:
- `play:CARD_ID:TARGET` — カードをプレイ（TARGET は `stage` または `backstage`）
- `select:CARD_ID` — スキル選択肢でカードを選択
- `select:CARD_ID:ZONE` — カード選択 + ゾーン選択の複合（例: カード003のスキル）
- `pass` — パス

CARD_ID はカード定義の ID（int）。実行時に instance_id に自動解決される。キューが尽きると手動操作に戻る。

### CPU 自動対戦（統合テスト）

`cpu=both` で両プレイヤーを CPU にし、`max_turns=N` でターン制限を設定できる。ゲーム終了後にプロセスが自動終了する。

```bash
make debug cpu=both max_turns=30
make debug test=9 cpu=both max_turns=50
make debug ARGS="p0=9 s1=r" cpu=both max_turns=30
make debug test=9 speed=3            # 3倍速で人間テスト
make debug cpu=both max_turns=30 speed=100  # 明示的速度指定
```

構造化ログが stdout に出力されるため、grep でイベントを検証できる:
```bash
make debug test=9 cpu=both max_turns=30 2>&1 | grep "SKILL_EFFECT"
```

- exit code 0: ゲームが正常終了（勝者決定）
- exit code 1: ターン制限到達（引き分け）

**機能実装時は、ユニットテストに加えて `cpu=both` による統合テストも実行し、ゲームがクラッシュせず完走することを確認すること。**
特にスキル実装後は、そのカードを含むプリセットで統合テストを行う:
```bash
make debug ARGS="p0=NEW_CARD_ID s1=r" cpu=both max_turns=50 speed=50
```

### 構造化ログ (GameLog)

ゲーム中の全イベントを構造化フォーマットで標準出力に出力する。

```
[00:03.241] [EVENT  ] SKILL_EFFECT player=0
[00:03.241] [ANIM   ] cutin_start skill=カオスそのもの nickname=ハコス
[00:03.850] [ANIM   ] cutin_end skill=カオスそのもの
[00:03.851] [ANIM   ] move_start iid=69 from=stage to=stage
[00:04.251] [ANIM   ] await_done
[00:04.260] [UI     ] refresh
```

カテゴリ: `SESSION`（システム層）, `STAGING`（演出層）, `ANIM`, `CHOICE`, `ACTION`, `UI`

## コーディング規約

### 変数宣言: `:=` 推論を避け、型を明示する

GDScript の `:=` は型なし `Array` / `Dictionary` のメソッド戻り値（`.duplicate()`, `.find()`, `.size()` 等）で `Variant` 推論エラーを起こしやすい。**原則として型を明示宣言する。**

```gdscript
# Good
var ids: Array = state.stages[p].duplicate()
var idx: int = deck.find(instance_id)
var count: int = hands[p].size()

# Bad — Variant 推論エラーの原因
var ids := state.stages[p].duplicate()
var idx := deck.find(instance_id)
```

`:=` を使ってよいケース: 右辺の型が明確なリテラル・コンストラクタ・型付きメソッド呼び出し。
```gdscript
var state := GameState.new()      # OK: コンストラクタ
var name := "test"                # OK: String リテラル
var count := 0                    # OK: int リテラル
```

## 設計上の重要事項

### カード設計: CardDef / CardInstance の分離

- **CardDef**: 静的なマスターデータ（名前、基本アイコン、基本スート、スキル定義）
- **CardInstance**: 実行時の個体（instance_id で一意識別）。バフ/デバフを Modifier として保存
- 複製スキルにより同一 card_id から複数インスタンスが存在しうるため、ゾーンには instance_id を格納
- 実効値 = 基本値 + Modifier による加減算

### Modifier とイベントフック

- バフ/デバフは保存型（Modifier オブジェクト）で管理
- 各 Modifier は `source_instance_id` と `persistent` フラグを持つ
- カードがゾーンを離脱した時、非永続 Modifier を自動クリーンアップ

### スキル解決スタック

- スキルの割り込み・カウンターに対応する LIFO（後入れ先出し）構造
- スキル発動 → トリガーチェック → カウンター可否の選択 → LIFO 順に解決
- PendingChoice はスキル解決専用。フェーズの行動選択は GameController が算出

### GameState / GameController の分離

- GameState: ピュアなデータ（状態保持のみ）
- GameController: ロジック層（行動算出、アクション適用、スキルスタック解決）

### 差分履歴

- 全ての状態変更を StateDiff（原子的変更）として記録
- GameAction が複数の StateDiff を束ね、プレイヤー行動の単位を形成
- 巻き戻し = diffs の逆順逆適用

### カードスキルのアーキテクチャ

- 約60枚の全カードがそれぞれユニークなスキルを持つ
- カードごとに固有のスキルコードが必要（共通化不可）
- スキル解決中にプレイヤーの入力待ち（選択）が発生しうる → PendingChoice で中断/再開
- スキルは3種類: Play Skill, Action Skill, Passive Skill

### ゲームフロー

- ターン制: ドロー → アクションフェーズ → プレイフェーズ → ターン終了チェック
- ライブ準備 → ショウダウンによるラウンド勝敗決定
- ラウンド勝利を規定数積み重ねてゲーム勝利

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gen0e0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gen0e0)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
