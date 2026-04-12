## tetris

> - **目標**: 高品質なUI/UXを持つモダンなテトリスWebアプリケーションの開発

# High-Quality Tetris Web Application Development Rules

## 🎮 プロジェクト概要
- **目標**: 高品質なUI/UXを持つモダンなテトリスWebアプリケーションの開発
- **対象**: デスクトップ・タブレット・モバイル対応のレスポンシブWebアプリ
- **コンセプト**: クラシックテトリスの魅力を保ちつつ、モダンな体験を提供

## 🛠️ 技術スタック

### フロントエンド
- **フレームワーク**: React 18+ with TypeScript
- **バンドラー**: Vite（高速開発環境）
- **スタイリング**: 
  - Tailwind CSS（ユーティリティファースト）
  - Framer Motion（アニメーション）
  - CSS Grid/Flexbox（レイアウト）
- **状態管理**: Zustand（軽量でシンプル）
- **オーディオ**: Web Audio API
- **Canvas/WebGL**: 必要に応じてパフォーマンス向上のため

### 開発ツール
- **Linting**: ESLint + Prettier
- **Testing**: Vitest + React Testing Library
- **Type Safety**: 厳格なTypeScript設定
- **Build**: Production最適化設定

## 📁 プロジェクト構造

```
src/
├── components/           # React コンポーネント
│   ├── Game/            # ゲームメインコンポーネント
│   ├── Board/           # ゲームボード関連
│   ├── UI/              # UI共通コンポーネント
│   └── Effects/         # ビジュアルエフェクト
├── hooks/               # カスタムReact hooks
├── stores/              # Zustand ストア
├── utils/               # ユーティリティ関数
├── types/               # TypeScript型定義
├── constants/           # ゲーム定数
├── styles/              # グローバルスタイル
└── assets/              # 静的アセット
```

## 🎨 UI/UX ガイドライン

### デザイン原則
1. **ミニマリズム**: クリーンで直感的なインターフェース
2. **アクセシビリティ**: WCAG 2.1 AA準拠
3. **レスポンシブ**: モバイルファースト設計
4. **パフォーマンス**: 60fps以上の滑らかな動作

### カラーパレット
- **プライマリ**: ネオン調のアクセントカラー
- **セカンダリ**: 落ち着いたダークテーマ
- **テトロミノ**: 各ピース固有の鮮やかな色

### アニメーション
- **落下**: イージング関数を使用した滑らかな動き
- **行消去**: 満足感のあるエフェクト
- **レベルアップ**: 祝福感のある演出
- **ゲームオーバー**: 適切なフィードバック

### インタラクション
- **キーボード**: WASD + Space/Arrow keys対応
- **タッチ**: スワイプ・タップジェスチャー
- **ゲームパッド**: 標準コントローラー対応
- **フィードバック**: 触覚・音響・視覚的反応

## 🎮 ゲーム機能仕様

### コア機能
- [ ] **Classic Mode**: 標準テトリス
- [ ] **Sprint Mode**: 40ライン消去タイムアタック
- [ ] **Zen Mode**: 時間無制限モード
- [ ] **Challenge Mode**: 特殊ルール

### ゲームシステム
- **SRS (Super Rotation System)**: 現代標準の回転システム
- **7-bag Randomizer**: テトロミノ生成アルゴリズム
- **Ghost Piece**: 落下予測表示
- **Hold System**: ピース保持機能
- **T-Spin**: 高度なテクニック対応

### スコアリング
- **Line Clear**: ライン数に応じたポイント
- **T-Spin Bonus**: 特殊技術ボーナス
- **Back-to-Back**: 連続高難度技術ボーナス
- **Perfect Clear**: 全消しボーナス

## 🔧 技術実装ガイドライン

### パフォーマンス
- **フレームレート**: 一定60fps維持
- **メモリ管理**: 不要オブジェクトの適切な解放
- **最適化**: requestAnimationFrame使用
- **Bundle Size**: 軽量化（<500KB gzipped）

### 状態管理
```typescript
interface GameState {
  board: CellType[][];
  currentPiece: Tetromino;
  nextPieces: Tetromino[];
  heldPiece: Tetromino | null;
  score: number;
  level: number;
  lines: number;
  gameStatus: 'playing' | 'paused' | 'gameOver';
}
```

### エラーハンドリング
- **Graceful Degradation**: 機能低下時の適切な対応
- **Error Boundaries**: React エラー境界
- **Logging**: 開発用ログシステム

## 🎵 オーディオシステム

### 音響設計
- **BGM**: ループ可能な背景音楽
- **SFX**: ピース落下・回転・消去音
- **Dynamic Audio**: ゲーム状況に応じた音響変化
- **Volume Control**: ユーザーによる音量調整

### 実装
- Web Audio API使用
- 音声ファイル最適化（OGG/WebM対応）
- 遅延最小化

## 📱 レスポンシブ対応

### ブレークポイント
- **Mobile**: <768px
- **Tablet**: 768px-1024px  
- **Desktop**: >1024px

### 各デバイス対応
- **タッチ操作**: スワイプ・タップ・長押し
- **画面回転**: ランドスケープ・ポートレート
- **PWA**: プログレッシブWebアプリ対応

## 🧪 テスト戦略

### ユニットテスト
- ゲームロジック（テトロミノ回転・移動・ライン消去）
- スコア計算
- 状態管理

### 統合テスト
- ユーザーインタラクション
- ゲームフロー
- データ永続化

### E2Eテスト
- 完全なゲームプレイフロー
- パフォーマンステスト

## 📊 データ管理

### ローカルストレージ
- ハイスコア
- 設定（音量・キーバインド）
- 統計情報

### データ構造
```typescript
interface UserData {
  highScores: Record<GameMode, number>;
  settings: UserSettings;
  statistics: GameStatistics;
}
```

## 🚀 デプロイメント

### ビルド最適化
- **Code Splitting**: 動的インポート
- **Tree Shaking**: 未使用コード除去
- **Asset Optimization**: 画像・音声圧縮
- **Caching Strategy**: 効率的キャッシュ戦略

### パフォーマンス目標
- **First Contentful Paint**: <1.5s
- **Largest Contentful Paint**: <2.5s
- **Cumulative Layout Shift**: <0.1
- **First Input Delay**: <100ms

## 🔒 品質保証

### コード品質
- **TypeScript**: 厳格型チェック
- **ESLint**: コード規約準拠
- **Prettier**: 一貫したフォーマット
- **Husky**: Pre-commit hooks

### セキュリティ
- **XSS Protection**: 適切なサニタイズ
- **CSP**: Content Security Policy設定
- **依存関係**: 定期的なセキュリティ監査

## 📝 開発進捗管理

### 進捗追跡システム
- **メインファイル**: [todo.md](mdc:todo.md) を使用した進捗管理
- **更新ルール**: 各タスク完了時に必ずチェックボックスにチェックを入れる
- **フェーズ管理**: 18フェーズに分けた段階的開発
- **依存関係**: 前のフェーズ完了後に次フェーズに進む

### 開発ワークフロー
1. **タスク開始前**: todo.mdで現在のフェーズと次のタスクを確認
2. **開発中**: 関連するファイルの参照（プロジェクト構造に従う）
3. **タスク完了後**: todo.mdのチェックボックスにチェック ✅
4. **フェーズ完了後**: 進捗状況セクションを更新
5. **問題発生時**: 開発メモセクションに記録

### 進捗レポート
- **完了率**: 自動計算（完了タスク/全タスク）
- **現在フェーズ**: 作業中のフェーズを明記
- **マイルストーン**: 次の重要な完了目標
- **ブロッカー**: 開発を阻害する問題の記録

### 品質チェックポイント
- **Phase完了時**: 該当機能の動作確認
- **統合時**: 既存機能との相互作用確認
- **テスト**: 各フェーズでの基本テスト実行

## 📈 今後の拡張性

### 機能拡張
- **マルチプレイヤー**: WebSocket対応
- **テーマシステム**: カスタマイゼーション
- **レプレイ機能**: ゲーム録画・再生
- **AI対戦**: コンピューター対戦

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mei0001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mei0001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
