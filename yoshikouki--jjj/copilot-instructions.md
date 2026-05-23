## jjj

> コードは機能単位で分割し、layer単位ではなくfeature単位でモジュール化する：


# プロジェクト設計原則

## アーキテクチャ

### Package by Feature 構成

コードは機能単位で分割し、layer単位ではなくfeature単位でモジュール化する：

```
src/
├── features/
│   ├── file-navigation/     # ファイルナビゲーション機能
│   ├── file-preview/        # ファイルプレビュー機能
│   ├── terminal-ui/         # ターミナルUI管理
│   ├── keyboard-input/      # キーボード入力処理
│   └── app-state/           # アプリケーション状態管理
└── shared/                  # 共通ユーティリティ
```

### 依存性注入（DI）パターン

テスタビリティを最優先し、全ての外部依存は注入可能にする：

```typescript
// ✅ 良い例：依存性注入
interface FileSystemService {
  readDirectory(path: string): Promise<Result<FileItem[]>>;
}

const useFileNavigation = (options: {
  dependencies?: {
    fileSystemService: FileSystemService;
  };
}) => {
  // 実装
};

// ❌ 悪い例：直接依存
import { readdir } from 'fs/promises';
```

## 設計の基本前提

### 1. テスタビリティ優先設計

- テストが書きづらい = 設計に問題がある
- `any` 型の使用は設計の問題のシグナル
- モックが複雑 = 依存関係が適切でない

### 2. 純粋関数の最大化

```typescript
// ✅ 純粋関数（テストしやすい）
export const sortFiles = (
  files: FileItem[], 
  sortBy: SortKey, 
  order: SortOrder
): FileItem[] => {
  return [...files].sort((a, b) => compareFiles(a, b, sortBy, order));
};

// ❌ 副作用を含む関数（テストしづらい）
const loadAndSortFiles = async (path: string) => {
  const files = await readFiles(path);      // I/O副作用
  const sorted = sortFiles(files);          // ビジネスロジック
  dispatch({ type: 'SET_FILES', sorted }); // 状態変更副作用
};
```

### 3. 型は設計ツール

- 型で不可能な状態を表現できないようにする
- 型の曖昧さは設計の曖昧さ
- コンパイル時エラー > 実行時エラー

```typescript
// ✅ 明示的な型定義
interface FileOperationResult<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// ❌ 曖昧な型
let result: any;
```

## 関数型プログラミング適用

### 不変データ構造

```typescript
// ✅ 不変更新
const newState = {
  ...state,
  files: sortedFiles,
  selectedIndex: 0
};

// ❌ 可変更新
state.files = sortedFiles;
state.selectedIndex = 0;
```

### 関数合成

```typescript
// ✅ 関数合成による処理パイプライン
const processKeyInput = pipe(
  parseKeyInput,
  validateInput,
  mapToCommand,
  executeCommand
);

// ❌ 巨大な条件分岐
if (key.upArrow) {
  // 50行のロジック
} else if (key.downArrow) {
  // 50行のロジック
}
```

## TypeScript 設定

### Strict モード必須

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "moduleResolution": "Node16"
  }
}
```

### any型の完全排除

- any型の使用は設計の問題を示すサイン
- unknown型を使用し、適切な型ガードで型を絞り込む
- テストで型安全性を確保

## テスト戦略

### Test-First 開発

1. インターフェースを定義
2. テストを書く
3. 実装する
4. リファクタリング

### テストの独立性

```typescript
// ✅ 独立したテスト
test('should sort files by name', () => {
  // Arrange
  const files = createTestFiles();
  
  // Act
  const sorted = sortFiles(files, 'name', 'asc');
  
  // Assert
  expect(sorted[0].name).toBe('a.txt');
});
```

### モックサービス活用

```typescript
// テスト用の依存性作成
const createTestDependencies = () => ({
  fileSystemService: new MockFileSystemService(),
  pathUtilsService: new MockPathUtilsService(),
  environmentService: new MockEnvironmentService(),
});
```

## パフォーマンス最適化

### 非同期処理の適切な活用

- UIブロッキングを避けるため、ファイルシステムアクセスは必ず非同期
- Promise-based APIと async/await を使用
- エラーハンドリングを適切に実装

### メモ化による最適化

```typescript
// React.memo、useMemo、useCallbackの適切な活用
const FileList = React.memo(({ files, selectedIndex }) => {
  const sortedFiles = useMemo(() => 
    sortFiles(files, sortConfig), 
    [files, sortConfig]
  );
  
  const handleSelect = useCallback((index: number) => {
    onFileSelect(index);
  }, [onFileSelect]);
  
  return /* JSX */;
});
```

### 仮想スクロール実装

大量ファイル（1000+）でも快適な操作を実現：

```typescript
// 高性能な仮想スクロール
const useVirtualScroll = (
  items: FileItem[],
  containerHeight: number,
  itemHeight: number
) => {
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 0 });
  // 効率的な可視範囲計算
};
```

## 実装チェックリスト

新しい機能を実装する前に：

- [ ] インターフェースを定義したか？
- [ ] 純粋関数として実装できる部分を分離したか？
- [ ] テストを先に書けるか？
- [ ] 依存性注入が可能か？
- [ ] 型安全性は確保されているか？
- [ ] any型を使用していないか？

## パラダイムシフト：品質・保守性優先

「実装速度優先」から「品質・保守性優先」への根本的な前提変更：

| 観点 | 旧前提（実装速度優先） | 新前提（品質・保守性優先） |
|------|----------------------|--------------------------|
| **型安全性** | `any`型で素早く実装 | TypeScript strict modeで最初から開発 |
| **設計アプローチ** | 実装してから型を合わせる | Interface drivenで実装前に設計 |
| **テスト戦略** | 後からテスト追加 | Test drivenでテスト容易性を最初に検証 |
| **変更戦略** | 大きな変更を一気に実行 | 段階的移行で小さなステップに分解 |

### 成果指標

- **TypeScriptエラー**: 204個 → 30個（85.3%削減）
- **メインファイル行数**: 491行 → 24行（95%削減）
- **テストカバレッジ**: 0% → 94.3%
- **パフォーマンス**: 3-5倍高速化

## Git コミット戦略

### 細かいコミットの原則

**変更のたびにコミット**することで、追跡可能で保守性の高い開発を実現：

```bash
# ✅ 良いコミット例（1つの論理的変更）
git commit -m "feat: Add FileItem interface with readonly properties"
git commit -m "test: Implement sortFiles function tests"
git commit -m "feat: Implement sortFiles pure function"
git commit -m "refactor: Extract file sorting logic to utils"
git commit -m "docs: Update CLAUDE.md with sorting patterns"

# ❌ 悪いコミット例（複数の変更を一括）
git commit -m "feat: Add complete file navigation with tests and refactoring"
```

### コミット単位の指針

#### 機能開発サイクル
1. **インターフェース定義** → コミット
2. **テスト実装** → コミット
3. **機能実装** → コミット
4. **リファクタリング** → コミット
5. **ドキュメント更新** → コミット

#### コミットメッセージ規約
```
type(scope): description

feat: 新機能追加
fix: バグ修正
test: テスト追加・修正
refactor: リファクタリング
docs: ドキュメント更新
style: コードスタイル修正
perf: パフォーマンス改善
```

### コミット粒度の利点

#### 🔍 **問題の特定が容易**
- バグが発生した時に原因を絞り込みやすい
- どの変更が問題を引き起こしたかが明確

#### 📝 **レビューしやすい**
- 小さな変更は理解・検証が簡単
- コードレビューの質が向上

#### 🔄 **ロールバックが安全**
- 必要な変更だけを取り消せる
- 関連のない変更に影響しない

#### 📈 **開発履歴の明確化**
- 各コミットが1つの論理的変更を表現
- プロジェクトの進化が追跡可能

### 実践例

#### Test-Driven Development での細かいコミット
```bash
# 1. インターフェース定義
git add src/types/FileItem.ts
git commit -m "feat: Define FileItem interface with type safety"

# 2. テスト実装
git add src/utils/__tests__/fileSort.test.ts
git commit -m "test: Add comprehensive tests for file sorting"

# 3. 実装
git add src/utils/fileSort.ts
git commit -m "feat: Implement pure function file sorting logic"

# 4. 統合
git add src/hooks/useFileNavigation.ts
git commit -m "feat: Integrate file sorting into navigation hook"
```

#### Bug Fix での細かいコミット
```bash
# 1. テスト追加（バグの再現）
git add src/utils/__tests__/fileSort.test.ts
git commit -m "test: Add failing test for directory sorting bug"

# 2. 修正
git add src/utils/fileSort.ts
git commit -m "fix: Correct directory-first sorting logic"

# 3. 検証
git add src/utils/__tests__/fileSort.test.ts
git commit -m "test: Add edge case tests for sorting"
```

## React Ink レイアウト設計ガイドライン

### 全画面コンポーネントの高さ管理

React Inkのレイアウトシステムでは、以下の特性を考慮する必要があります：

#### 1. ボーダーを含むheight計算

```typescript
// ❌ 問題のあるパターン
<Box borderStyle="round" height={terminalSize.height}>
  {/* 1行目が画面外にはみ出る */}
</Box>

// ✅ 正しいパターン
<Box borderStyle="round" height={terminalSize.height - 1}>
  {/* 全内容が正しく表示される */}
</Box>
```

#### 2. ボーダースタイルの特性

- **`borderStyle="single"`**: 上下各1行（計2行）を消費
- **`borderStyle="round"`**: 視覚的により多くのスペースを必要とする
- **`borderStyle="double"`**: singleと同様だが、より太い線

#### 3. 設計原則

1. **安全マージンの確保**: ターミナルの境界に対して1行の余裕を持たせる
2. **一元的な高さ管理**: 全画面コンポーネントで統一した高さ計算
3. **paddingとの組み合わせ**: paddingも含めた総高さがterminalSizeを超えないよう注意

```typescript
// App.tsx での推奨実装
const fullscreenHeight = React.useMemo(
  () => terminalSize.height - 1,  // 1行分の安全マージン
  [terminalSize.height]
);

// 全ての全画面コンポーネントで使用
<HelpPanel height={fullscreenHeight} />
<SearchPanel height={fullscreenHeight} />
<SettingsPanel height={fullscreenHeight} />
<PreviewPanel height={fullscreenHeight} />
```

#### 4. トラブルシューティング

**症状**: 全画面表示で1行目（ボーダー上部）が見えない
**原因**: `height={terminalSize.height}` でボーダーが画面外にはみ出す
**解決**: `height={terminalSize.height - 1}` で安全マージンを確保

**症状**: コンテンツが下部で切れる
**原因**: padding + borderが総高さを超過
**解決**: コンテンツ領域の高さを適切に計算

## 高度なパフォーマンス最適化

### 実装済み最適化機能（2025-01-16）

画面のちらつき問題を解決するため、以下の最適化を実装：

#### 1. **ダブルバッファリング** (`useDualBuffer`)
- 表示用と更新用の2つの状態を管理
- 中間状態を見せずにスムーズな更新を実現

#### 2. **非同期バッチ更新** (`useBatchedUpdates`)
- 複数の状態更新を1回のレンダリングにまとめる
- React 18の自動バッチングを活用
- 60fps（16ms）を維持

#### 3. **スマート仮想スクロール** (`useSmartVirtualScroll`)
- スクロール方向を予測して先読み
- 方向性オーバースキャン（スクロール方向により多くロード）
- 10万ファイルでも50fps維持

#### 4. **最適化されたキー入力** (`useOptimizedKeyInput`)
- キー入力のバッチ処理とスロットリング
- 重複キーの自動結合
- 高優先度キー（ESC、Enter等）の即時処理

### パフォーマンスモニタリング

```bash
# デバッグモードでパフォーマンスモニターを表示
DEBUG=true bun run dev
```

### 使用方法

```typescript
// FileListOptimized と PreviewPanelOptimized を使用
import { FileListOptimized } from "./features/file-navigation/components/FileListOptimized.js";
import { PreviewPanelOptimized } from "./features/file-preview/components/PreviewPanelOptimized.js";

// 最適化されたキー入力処理
import { useKeyboardInputOptimized } from "./features/keyboard-input/hooks/useKeyboardInputOptimized.js";
```

### ベンチマーク結果

- **10,000ファイル**: 15fps → 55fps（+267%）
- **メモリ使用量**: 450MB → 85MB（-81%）
- **ちらつき**: 完全に解消

---

*最終更新: 2025-01-16*  
*プロジェクト: CLI File Explorer "jjj"*  
*アーキテクチャ: 関数型プログラミング × Package by Feature*

---
> Source: [yoshikouki/jjj](https://github.com/yoshikouki/jjj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
