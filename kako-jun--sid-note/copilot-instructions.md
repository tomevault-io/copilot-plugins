## sid-note

> sid-noteは、ベースギターのフィンガリングを楽譜より具体的に表示するWebアプリケーション。

# sid-note プロジェクト - Claude開発ドキュメント

## 🎯 プロジェクトビジョン

sid-noteは、ベースギターのフィンガリングを楽譜より具体的に表示するWebアプリケーション。
TAB譜よりも詳細で、音楽理論に基づいた視覚的なガイドを提供する。

## 📐 アーキテクチャ設計

### 技術スタック

**フロントエンド（現在）:**
- Next.js 15.3.2
- React 19.0.0
- TypeScript 5
- Tailwind CSS 4

**音楽理論エンジン（移行中）:**
- **Phase 1（完了）:** TypeScript実装（人力で書いた最後のバージョン）
- **Phase 2（実施中）:** Rust + WASM実装（高速化・型安全性向上）

### レイヤー構成

```
┌─────────────────────────────────────────────┐
│         UI Layer (React Components)          │
│  - Keyboard, Staff, CircleOfFifths, etc.    │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      Music Theory Layer (WASM/Rust)         │
│  - Note, Chord, Scale, Harmony analysis     │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│     Instrument Layer (WASM/Rust)            │
│  - Bass fretboard position calculation      │
│  - Fingering pattern generation             │
└─────────────────────────────────────────────┘
```

## 🎸 ドメインモデル

### コア概念

1. **Note（音符）**
   - Pitch（音高）: C2, D＃3, E♭4など
   - Enharmonic equivalents（異名同音）: C＃ = D♭
   - Staff line position（五線譜位置）

2. **Chord（和音）**
   - Root note（根音）
   - Quality（品質）: Major, Minor, Dominant 7th, etc.
   - Intervals（音程）: 1, 3, 5, 7, etc.
   - Fretboard positions（フレット位置）

3. **Scale（音階）**
   - Type: Major, Minor, Modes
   - Diatonic chords（ダイアトニックコード）
   - Circle of fifths position（五度圏上の位置）

4. **Harmony（和声）**
   - Functional harmony（機能和声）: Ⅰ, Ⅱ, Ⅲ, Ⅳ, Ⅴ, Ⅵ, Ⅶ
   - Cadences（カデンツ）: Perfect, Plagal, Deceptive, Half
   - Chord progressions（コード進行）

### 楽器固有の機能

**Bass Guitar（4弦ベース）:**
- Standard tuning: E1-A1-D2-G2
- Fret range: 0-24
- String-to-fret position mapping

## 🚀 Rust/WASM移行戦略

### なぜRust + WASMか？

1. **パフォーマンス**: 音楽理論計算の高速化
2. **型安全性**: コンパイル時のエラー検出
3. **ポータビリティ**: 将来的に他のプラットフォームでも利用可能
4. **メモリ安全性**: メモリリーク・不正アクセスの防止

### 移行フェーズ

#### ✅ Phase 1: 基盤構築（完了）
- [x] Rustプロジェクト初期化
- [x] WASM bindings設定
- [x] モジュール構成設計
- [x] ビルドシステム構築

#### ✅ Phase 2: 機能移植（完了）
- [x] Core module（Note機能）
- [x] Chord module（コード解析・ポジション計算）
- [x] Scale module（スケール・ダイアトニックコード）
- [x] Harmony module（機能和声・カデンツ）
- [x] Utils module（クロマチック判定・別表記）

#### ✅ Phase 3: テスト・検証（完了）
- [x] Rustテスト作成（21個）
- [x] TypeScriptテスト作成（34個）
- [x] デグレチェック実施
- [x] バグ発見・修正

#### ⏸️ Phase 4: WASM統合（保留）
- [ ] wasm-packでビルド
- [ ] Next.jsからWASM呼び出し
- [ ] パフォーマンステスト
- [ ] 段階的な移行

#### 📅 Phase 5: 最適化（未着手）
- [ ] WASMバンドルサイズ最適化
- [ ] 並列処理の導入
- [ ] キャッシング戦略

#### 📅 Phase 6: ライブラリ分離（未着手）
- [ ] 独立リポジトリへ分離
- [ ] NPMパッケージ化
- [ ] ドキュメント整備
- [ ] CI/CD構築

## 🐛 発見したバグ

### TypeScript実装のバグ

**Bug #1: 異名同音判定の不具合**
- **場所**: `src/utils/noteUtil.ts:174-195` `comparePitch()`
- **症状**: `C＃2`と`D♭2`が異なると判定される（本来は同じ音）
- **原因**: 文字列比較しているため、異名同音を判定できない
- **修正**: Rust実装では半音インデックスで比較（すでに修正済み）
- **影響**: 現在のアプリで異名同音を使った場合、誤った判定がされる可能性

```typescript
// 現在の実装（バグあり）
return p1.names.some((n1) => p2.names.includes(n1));
// C＃ と D♭ は文字列として異なるのでfalse

// 修正案（Rustでは実装済み）
// 半音インデックスで比較すべき
```

**対応方針**:
- Rust実装は正しい動作をする
- TypeScript実装は互換性のため残すが、修正するかは今後検討

## 📊 プロジェクト統計

### コード規模

| 言語 | ファイル数 | 行数（概算） |
|------|-----------|------------|
| TypeScript/TSX | 30+ | 3,500+ |
| Rust | 15 | 1,700+ |
| テスト（TS） | 1 | 230+ |
| テスト（Rust） | 15（組み込み） | 300+ |

### 機能カバレッジ

| カテゴリ | 関数数 | Rust移植 | テストカバレッジ |
|---------|--------|----------|----------------|
| Note | 4 | ✅ 100% | ✅ 100% |
| Chord | 8 | ✅ 100% | ✅ 62.5% |
| Scale | 4 | ✅ 100% | ✅ 100% |
| Harmony | 7 | ✅ 100% | ✅ 57% |
| Utils | 3 | ✅ 100% | ✅ 100% |
| **合計** | **26** | **✅ 100%** | **✅ 80%** |

## 🎨 UI/UX設計

### 主要コンポーネント

1. **Keyboard（鍵盤）**: `src/components/performance/Keyboard.tsx`
   - 音符の視覚的表示
   - インタラクティブな操作

2. **Staff（五線譜）**: `src/components/performance/Staff.tsx`
   - 楽譜表示
   - 音高の視覚化

3. **CircleOfFifths（五度圏）**: `src/components/track/CircleOfFifths.tsx`
   - キーの関係性表示
   - 調性の把握

4. **DiatonicChordTable**: `src/components/track/DiatonicChordTable.tsx`
   - ダイアトニックコード表
   - 機能和声の表示

## 🔧 開発ワークフロー

### ローカル開発

```bash
# Next.js開発サーバー
npm run dev

# TypeScriptテスト
npm test

# Rustテスト
cd rust-music && cargo test

# WASMビルド（Phase 4以降）
cd rust-music && wasm-pack build --target web
```

### コードスタイル

**TypeScript:**
- ESLint + Next.js config
- 関数型プログラミング優先
- コンポーネントは小さく保つ

**Rust:**
- `cargo fmt`でフォーマット
- `cargo clippy`でリント
- ドキュメントコメント必須（`///`）

## 📚 技術的負債

1. **テストカバレッジ不足**
   - Phase 2完了時点で初めてテスト追加
   - 既存のReactコンポーネントにテストなし

2. **型安全性**
   - YAMLデータの型チェックが弱い
   - Zodスキーマはあるが実行時のみ

3. **パフォーマンス**
   - 大量のコードポジション計算が重い可能性
   - 最適化前の実装

4. **ドキュメント**
   - ユーザー向けドキュメント不足
   - APIドキュメント未整備

## 🗺️ ロードマップ

### 短期（〜1ヶ月）
- [ ] WASM統合完了
- [ ] TypeScriptバグ修正
- [ ] テストカバレッジ向上

### 中期（〜3ヶ月）
- [ ] パフォーマンス最適化
- [ ] ユーザードキュメント作成
- [ ] モバイル対応改善

### 長期（〜6ヶ月）
- [ ] 音楽理論ライブラリの独立化
- [ ] 他の楽器対応（ギター、ウクレレ等）
- [ ] コミュニティ機能追加

## 🔐 セキュリティ

- XSS対策: Reactのデフォルトエスケープ
- YAML解析: js-yamlの安全なパーサー使用
- 依存関係: npm audit で定期チェック

## 📝 参考リソース

### 音楽理論
- [Music Theory for Computer Musicians](https://www.musictheory.net/)
- [Open Music Theory](https://viva.pressbooks.pub/openmusictheory/)

### Rust/WASM
- [Rust and WebAssembly Book](https://rustwasm.github.io/docs/book/)
- [wasm-bindgen Guide](https://rustwasm.github.io/wasm-bindgen/)

### 既存ライブラリ
- [rust-music-theory](https://github.com/ozankasikci/rust-music-theory)
- [kord (Rust/WASM)](https://github.com/twitchax/kord)

---

**最終更新**: 2025-11-16
**メンテナー**: Claude + kako-jun
**ステータス**: Phase 3完了、Phase 4保留中

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kako-jun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kako-jun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
