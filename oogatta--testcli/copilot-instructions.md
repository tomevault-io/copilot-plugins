## testcli

> このドキュメントは、Robert C. Martin（Uncle Bob）の**4層オニオンアーキテクチャ**とVladimir Khorikivの**単体テストの原則、実践、パターン**に従って構築されたTypeScript製testcliアプリケーションのアーキテクチャとテストガイドラインを説明しています。**関数モジュール**、**Command Query Separation (CQS)**、**Dependency Inversion Principle (DIP)** に重点を置いた、堅牢でテスト可能なEthereumトランザクション構築・署名・送信CLIアプリの作成を目標としています。

# testcli: アーキテクチャとテストガイドライン

このドキュメントは、Robert C. Martin（Uncle Bob）の**4層オニオンアーキテクチャ**とVladimir Khorikivの**単体テストの原則、実践、パターン**に従って構築されたTypeScript製testcliアプリケーションのアーキテクチャとテストガイドラインを説明しています。**関数モジュール**、**Command Query Separation (CQS)**、**Dependency Inversion Principle (DIP)** に重点を置いた、堅牢でテスト可能なEthereumトランザクション構築・署名・送信CLIアプリの作成を目標としています。

## プロジェクト概要

- **目的**: CLIアプリ（`testcli`）で、入力（例：`--to`, `--amount`, `--privateKey`）を受け取り、Ethereumトランザクションを構築・署名し、（最終的に）テストネットに送信する。
- **技術スタック**: TypeScript, Node.js, ethers.js, Vitest, commander.js。

## アーキテクチャ: 4層オニオンアーキテクチャ

このプロジェクトはUncle BobのClean Architectureに従い、`enterprise-business-rules`、`application-business-rules`、`interface-adapter`、`frameworks`の4層で構成されています。各層には特定の役割があり、依存関係は内側に向かって流れます（外側の層が内側の層に依存）。

interface-adapaters以上では、Nodeには依存しないが、TypeScript(JavaScript)の提供するオブジェクト等には依存してよい。

### 1. enterprise-business-rules（エンタープライズビジネスルール層）

- **役割**: 外部システムに依存しない純粋なビジネスロジック。ビジネスルール（例：トランザクション検証）を持つコアエンティティを含む。
- **例**: `Transaction`エンティティはアドレス、金額、ガス制限を検証する。
- **構成**: エンティティクラスとそのデータ構造インターフェースを含む。
- **重要な原則**: 外部依存なし（例：ethers.jsなし）。純粋なTypeScriptロジック。
- **テスト**: Vitestによる単体テスト、全てのエッジケース（例：無効入力）をカバー。

### 2. application-business-rules（アプリケーションビジネスルール層）

- **役割**: アプリケーション固有のロジック、ユースケースとアプリケーションモデルを含む。ビジネスルールをワークフローに編成。
- **例**: `buildTransaction`（クエリ）や`signTransaction`（コマンド）などのユースケース。
- **構成**: ユースケース関数、エンティティクラス、インターフェース定義を含む。
- **重要な原則**: シンプルさとテスタビリティのため**関数モジュール**（クラスではない）を使用。**CQS**に従う:
  - クエリ（例：`buildTransaction`）: データを返し、副作用なし。
  - コマンド（例：`signTransaction`）: 副作用を起こし、最小限のデータを返す。
- **テスト**: Vitestによる単体テスト、`application-business-rules/interfaces`の依存関係（例：`Provider`、`Wallet`）をモック。

### 3. interface-adapters（インターフェースアダプター層）

- **役割**: 内部ロジックと外部システムを橋渡しする。外部依存（例：ethers.js）の型をドメインモデルに変換するアダプターを含む。
- **例**: `adaptExternalTransactionReceiptToDomain`は外部ライブラリの`TransactionReceipt`を内部の`TransactionReceipt`エンティティに変換。
- **構成**: 型変換アダプター関数と外部システムのインターフェース定義を含む。
- **重要な原則**: 外部ライブラリに直接依存せず、独自のインターフェースを定義して変換処理を行う。

### 4. frameworks（フレームワーク層）

- **役割**: 外部システム（例：ethers.js、Infura、Node.js API）と相互作用。`application-business-rules/interfaces`のインターフェースを実装し、CLIエントリーポイントをホスト。
- **例**: `EthersInfuraProvider`は`Provider`を実装；`EthersWallet`は`Wallet`を実装；CLIは入力を解析。
- **構成**: CLIエントリーポイント、外部ライブラリの実装クラス、ユーティリティ関数を含む。
- **重要な原則**: 管理されていない依存関係（ethers.js、Infura）の薄いラッパー。最小限のロジック、ビジネスルールなし。
- **テスト**: 最小限の単体テスト（Khorikivの「管理されていない依存関係」）。モックされたインターフェース経由で統合テストでカバー。

## ディレクトリ構造

```plaintext
src/
├── enterprise-business-rules/
│   ├── entities/
│   ├── interfaces/
│   ├── errors/
│   └── __tests__/
├── application-business-rules/
│   ├── entities/
│   ├── usecases/
│   ├── interfaces/
│   ├── errors/
│   └── __tests__/
├── interface-adapters/
│   ├── adapters/
│   ├── repositories/
│   ├── errors/
│   └── __tests__/
└── frameworks/
    ├── commanders/
    ├── ethers/
    └── node/
```

## テストガイドライン（Khorikivの原則）

テストはKhorikivの『単体テストの原則、実践、パターン』に従い、堅牢で保守性の高いテストを確保します。

### 単体テスト

- **スコープ**: `enterprise-business-rules`（エンティティ）、`application-business-rules`（ユースケース）、`interface-adapters`（アダプター）。
- **原則**:
  - **ビジネス価値をテスト**: 高価値ロジック（例：無効トランザクションの防止）に焦点。
  - **管理されていない依存関係をモック**: Vitestの`vi.mock`を使用して`application-business-rules`インターフェース（例：`Provider`、`Wallet`）をモック。
  - **スタブとの相互作用は検証しない**: モック呼び出しではなく、出力（クエリ）または副作用（コマンド）を検証。
  - **CQS**: クエリは戻り値をテスト；コマンドは副作用をテスト。
- **例**（`src/application-business-rules/__tests__/build-transaction.test.ts`より）:

  ```typescript
  import { vi, describe, it, expect } from 'vitest'
  import { buildTransaction } from '../usecases/build-transaction'
  import { Provider } from '../interfaces/provider'

  describe('buildTransaction', () => {
    const mockProvider: Provider = {
      getBalance: vi.fn().mockResolvedValue(BigInt('1000000000000000000')),
      getFeeData: vi
        .fn()
        .mockResolvedValue({ maxFeePerGas: BigInt('1000'), maxPriorityFeePerGas: BigInt('100') }),
      getTransactionCount: vi.fn().mockResolvedValue(0),
      broadcastTransaction: vi.fn().mockResolvedValue({ hash: '0x123' }),
      getTransactionReceipt: vi.fn().mockResolvedValue(null),
    }
    it('throws for invalid address', async () => {
      await expect(buildTransaction('invalid', '1.5', mockProvider, '0x123')).rejects.toThrow(
        'Invalid address'
      )
    })
  })
  ```

- **カバレッジ目標**: `enterprise-business-rules`、`application-business-rules`、`interface-adapters`で70%（`npm run coverage`で実行）。

### 統合テスト

- **スコープ**: `enterprise-business-rules`、`application-business-rules`、`interface-adapters`の統合を検証（`frameworks`を除く）。
- **原則**:
  - **実装ではなくインターフェースをモック**: `frameworks`の`EthersInfuraProvider`や`EthersWallet`ではなく、`application-business-rules/interfaces`の`Provider`と`Wallet`をモック。
  - **管理されていない依存関係**: `frameworks`層（ethers.js、Infura）は信頼できないものとして扱い、常にモック。
  - **統合フローをテスト**: ユースケースとエンティティが連携して動作することを確認（例：トランザクションの構築と署名）。
- **例**（`tests/integration/transaction.integration.test.ts`より）:

  ```typescript
  import { vi, describe, it, expect } from 'vitest'
  import { buildTransaction } from '../../src/application-business-rules/usecases/build-transaction'
  import { signTransaction } from '../../src/application-business-rules/usecases/sign-transaction'
  import { Provider } from '../../src/application-business-rules/interfaces/provider'
  import { Wallet } from '../../src/application-business-rules/interfaces/wallet'

  describe('Transaction Integration', () => {
    const mockProvider: Provider = {
      getBalance: vi.fn().mockResolvedValue(BigInt('1000000000000000000')),
      getFeeData: vi
        .fn()
        .mockResolvedValue({ maxFeePerGas: BigInt('1000'), maxPriorityFeePerGas: BigInt('100') }),
      getTransactionCount: vi.fn().mockResolvedValue(0),
      broadcastTransaction: vi.fn().mockResolvedValue({ hash: '0x123' }),
      getTransactionReceipt: vi.fn().mockResolvedValue(null),
    }
    const mockWallet: Wallet = {
      address: '0x1234567890123456789012345678901234567890',
      signTransaction: vi.fn().mockResolvedValue('0xSignedTx'),
    }

    it('builds and signs transaction', async () => {
      const tx = await buildTransaction(
        '0x1234567890123456789012345678901234567890',
        '1.5',
        mockProvider,
        mockWallet.address
      )
      const signedTx = await signTransaction(tx, mockWallet)
      expect(signedTx).toBe('0xSignedTx')
    })
  })
  ```

- **注記**: `frameworks`層は直接テストせず、そのインターフェースをモックして外部動作をシミュレート。

## 主要な設計決定

1. **ユースケースの関数モジュール**:
   - ユースケース（`build-transaction.ts`、`sign-transaction.ts`）は、シンプルさとテスタビリティのため、クラスではなく関数モジュールとして実装。
   - 理由: ボイラープレートを削減し、TypeScriptの軽量スタイルに合致し、CLIワークフローに適している。
2. **`frameworks`層のCLIエントリーポイント**:
   - `src/frameworks/node/index.ts`に配置し、引数解析に`commander.js`を使用。
   - コマンド定義は`src/frameworks/commanders/`に分離。
   - 理由: CLIは外部入力（`process.argv`、環境変数）を処理し依存関係を配線するため、`frameworks`層の役割に適合。
3. **外部依存関係のDIP**:
   - `Provider`と`Wallet`インターフェース（`application-business-rules/interfaces`内）がethers.jsとInfuraを抽象化。
   - 実装（`EthersInfuraProvider`、`EthersWallet`）は`frameworks`にあり、DIPを通じて注入。
4. **CQS**:
   - クエリ（例：`buildTransaction`）はデータを返す；コマンド（例：`signTransaction`）は副作用を起こす。
   - テストはCQSを反映：クエリテストは出力をチェック、コマンドテストは副作用をチェック。
5. **`frameworks`の最小限テスト**:
   - `frameworks`層（例：`EthersInfuraProvider`、`EthersWallet`）は管理されていない依存関係（ethers.js、Infura）の薄いラッパー。
   - `frameworks`の単体テストなし；統合テストでモックされたインターフェース経由でカバー。

## ガイドライン

### 技術文書の言語使い分け

- **日本語メイン**: ドキュメント全体は日本語で記述し、日本人開発者にとって理解しやすくする。
- **技術用語**: 重要な概念は日本語+英語併記（例：「4層オニオンアーキテクチャ」「CQS（Command Query Separation）」「DIP（Dependency Inversion Principle）」）。
- **コード例**: 英語のまま維持（国際的な慣例に従い、コメントやファイル名は英語）。
- **専門概念**: 正確な日本語用語を使用（例：「依存関係注入」「単体テスト」「統合テスト」「ビジネスルール」）。
- **ファイル名・パス**: 英語のまま（実際のコードベースと一致させる）。

### コード構成ガイドライン

- **import文の並び順**: Clean Architectureの抽象度に従って整理し、コメントで区分:
  1. `// enterprise-business-rules`
  2. `// application-business-rules`
  3. `// interface-adapters`
  4. `// frameworks`
  5. `// npm packages`
- **import文のパス表記**: 必ず`@/`エイリアスを使用してsrcディレクトリを参照する（例：`@/enterprise-business-rules/entities/transaction`）。相対パス（`../../../src/`など）は使用しない。

### 開発ガイドライン

- **4層構造に従う**: エンティティは`enterprise-business-rules`、ユースケースとインターフェースは`application-business-rules`、アダプターは`interface-adapters`、外部実装/CLIは`frameworks`に配置。
- **関数モジュールを使用**: ユースケースやアダプタはクラスではなく関数（例：`export async function buildTransaction(...)`）として実装。
- **暗黙のグローバル変数禁止**: 依存関係は常に関数パラメータとして明示的に渡す。関数内で暗黙的にアクセスされるグローバル変数や定数は使用しない。全ての依存関係は注入する。
- **Khorikivのテスト原則に従う**:
  - 単体テスト: `frameworks`実装ではなく、`application-business-rules`インターフェース（例：`Provider`、`Wallet`）をモック。
  - 統合テスト: `Provider`と`Wallet`をモックし、`enterprise`と`application`層の統合をテスト。
  - モックの相互作用は検証しない（例：`getGasPrice`が呼ばれたかをチェックしない）。
- **CQSを遵守**: ユースケースとテストでクエリ（データを返す）とコマンド（副作用）を分離。
- **層依存ルール（DIP強制）**:
  - `enterprise-business-rules`層: 自分の層のみからインポート可能
  - `application-business-rules`層: 自分の層と`enterprise-business-rules`からインポート可能
  - `interface-adapters`層: 自分の層、`application-business-rules`、`enterprise-business-rules`からインポート可能
  - `frameworks`層: 任意の層からインポート可能（制限なし）
- **`frameworks`層のimportルール**: エントリーポイント（CLI）以外の`frameworks`層実装では、business rules 2層（`enterprise-business-rules`、`application-business-rules`）から型定義をインポートする場合は極力`import type`を使用する。
- **`frameworks`テストを最小化**: `frameworks`を信頼できないものとして扱い、テストではそのインターフェースをモック。

## 実践的テスト構造化ガイドライン

このセクションでは、今回のプロジェクトで実践したテスト構造化の具体的な方針を示します。

### テストファイルの構造化パターン

各テストファイルは以下の構造に従って組織化する：

```typescript
describe('UseCase/Entity', () => {
  describe('successful scenarios', () => {}) // 正常系のテスト
  describe('business rules validation', () => {}) // ビジネスルールのテスト
  describe('edge cases', () => {}) // 境界値・特殊ケース
  describe('error handling', () => {}) // エラーハンドリング
  describe('delegation/transformation', () => {}) // 委譲・変換処理（必要に応じて）
})
```

### Managed vs Unmanaged Dependencies の判定と対応

**Managed Dependencies（実物使用）:**

- `enterprise-business-rules`層のエンティティ（例：`Transaction`）
- `interface-adapters`層のアダプター（例：`dataToTransaction`, `transactionToData`）
- `application-business-rules`層のエンティティ（例：`TransactionReceipt`）
- プロジェクト内で管理・制御可能なすべてのコード

**Unmanaged Dependencies（モック使用）:**

- `frameworks`層の実装（例：`EthersInfuraProvider`, `EthersWallet`, `NodeSleeper`）
- 外部ライブラリに直接依存するコンポーネント
- ネットワーク、ファイルシステム、時間に依存するコンポーネント

### ユースケース別テストパターン

#### 1. シンプルな委譲パターン（例：`getFees`, `getNonce`）

```typescript
describe('getFees', () => {
  describe('successful fee retrieval', () => {
    it('returns fee data with both values', async () => {
      // モックの設定
      // 実行
      // 戻り値の検証
      // モック呼び出しの検証（回数・引数）
    })
  })

  describe('edge cases', () => {
    it('handles null values', async () => {})
    it('handles very large values', async () => {})
  })

  describe('error handling', () => {
    it('propagates provider errors', async () => {})
  })

  describe('delegation behavior', () => {
    it('delegates directly without modification', async () => {
      // 同じ参照が返されることを確認（toBe使用）
    })
  })
})
```

#### 2. 変換処理のある委譲（例：`sendSignedTransaction`）

```typescript
describe('sendSignedTransaction', () => {
  describe('transformation behavior', () => {
    it('transforms provider response to extract hash', async () => {
      // オブジェクト → 文字列への変換を検証
      expect(typeof result).toBe('string')
      expect(result).toBe('0xtransformed_hash')
    })
  })
})
```

#### 3. 複雑な統合パターン（例：`buildTransaction`, `signTransaction`）

```typescript
describe('buildTransaction', () => {
  describe('successful transaction building', () => {
    it('builds transaction with correct parameters', async () => {
      // 実際のadapterとentityを使用
      const transactionAdapter = dataToTransaction(mockAddressValidator)
      const result = await buildTransaction(...)

      // 結果のプロパティを検証
      expect(result.chainId).toBe(11155111)

      // unmanagedな依存関係のモック検証
      expect(mockAddressValidator.validate).toHaveBeenCalledTimes(1)
    })
  })
})
```

#### 4. 複雑なビジネスロジック（例：`confirmTransaction`）

```typescript
describe('confirmTransaction', () => {
  describe('timeout behavior', () => {
    it('throws error after 20 failed attempts', async () => {
      // 複数回の呼び出しをモック
      // リトライロジックの検証
      expect(mockProvider.getTransactionReceipt).toHaveBeenCalledTimes(40)
      expect(mockSleeper.sleep).toHaveBeenCalledTimes(40)
    })
  })
})
```

### モック相互作用の検証パターン

**Khorikivの「モックしたら相互作用を検証する」原則を適用：**

```typescript
// 必須の検証項目
expect(mockFunction).toHaveBeenCalledWith(expectedArgs) // 引数の検証
expect(mockFunction).toHaveBeenCalledTimes(expectedCount) // 呼び出し回数の検証

// エラーケースでも必ず検証
await expect(useCase()).rejects.toThrow('Expected Error')
expect(mockFunction).toHaveBeenCalledTimes(1) // エラー時も呼び出し回数を確認
```

### ビジネスルールテストの重点項目

1. **固定値の検証**：
   - Sepoliaチェーン（11155111）
   - EIP1559タイプ（2）
   - ガス制限（21000）
   - 待機時間（15000ms）

2. **バリデーション**：
   - アドレス検証
   - 金額の正数チェック
   - 残高不足チェック

3. **境界値**：
   - 0, null, MAX値
   - リトライ回数の上限
   - タイムアウト条件

### エラーハンドリングテストの網羅性

```typescript
describe('error handling', () => {
  it('propagates network errors', async () => {}) // ネットワーク系
  it('propagates validation errors', async () => {}) // バリデーション系
  it('propagates business rule violations', async () => {}) // ビジネスルール系
  it('propagates external service errors', async () => {}) // 外部サービス系
})
```

### テスト命名規則

- **成功ケース**: 「動詞 + 期待する結果」（例：`returns nonce for valid address`）
- **エラーケース**: 「動詞 + エラー条件」（例：`throws error for invalid address`）
- **ビジネスルール**: 「always/enforces + ルール内容」（例：`always uses Sepolia chain ID`）
- **境界値**: 「handles + 特殊条件」（例：`handles very large values`）

### テストの段階的複雑さ

1. **レベル1**: シンプルな委譲（`getFees`, `getNonce`）
2. **レベル2**: 変換処理のある委譲（`sendSignedTransaction`）
3. **レベル3**: ビジネスロジック付き（`checkBalance`）
4. **レベル4**: 複雑な統合（`buildTransaction`）
5. **レベル5**: 最高複雑度（`signTransaction`, `confirmTransaction`）

各レベルで前のレベルのパターンを応用し、段階的に複雑さを増していく。

### テスト保守性のための原則

1. **DRY原則の適用**: 共通のモックやデータは`beforeEach`で初期化
2. **明確な分離**: 各テストケースは独立して実行可能
3. **分かりやすい命名**: テストケース名から期待動作が明確
4. **適切なグルーピング**: 関連するテストは`describe`でグループ化
5. **コメントの活用**: 複雑なロジックには説明コメントを追加

この構造化により、テストが仕様書として機能し、リファクタリング時の安全網となり、新機能追加時の指針となる。

## Orchestratorパターン（複合ワークフローの統合）

複雑なワークフローをテスト可能にするため、このプロジェクトでは**Orchestratorパターン**を採用しています。これは`frameworks`層のCLIエントリーポイントに含まれていたビジネスロジックをテスト可能な形に抽出する手法です。

### Orchestratorパターンの目的

1. **テスト可能性の向上**: CLIエントリーポイント（`frameworks/node/index.ts`）のビジネスロジックをテスト可能なユニットに抽出
2. **責任の分離**: CLIは引数解析とエラーハンドリングのみ、ビジネスワークフローはOrchestratorが担当
3. **ワークフロー全体の統合テスト**: 複数のユースケースを連携させたエンドツーエンドの処理をテスト

### Orchestratorの配置と構成

**配置**: `src/application-business-rules/orchestrators/`

**構成要素**:

- **SendTransactionOrchestrator**: 完全な送信ワークフロー（残高確認 → ナンス取得 → 手数料取得 → トランザクション構築 → 署名 → 送信 → 確認）
- **SignTransactionOrchestrator**: 署名のみのワークフロー（トランザクション構築 → 署名）
- **依存関係ファクトリー**: `frameworks/factories/dependency-factory.ts`で実際のframeworks実装を生成

### Orchestratorインターフェース設計

```typescript
// SendTransactionOrchestrator例
export interface SendTransactionDependencies {
  provider: Provider // unmanaged dependency (mocked)
  wallet: Wallet // unmanaged dependency (mocked)
  sleeper: Sleeper // unmanaged dependency (mocked)
  addressValidator: AddressValidator // unmanaged dependency (mocked)
}

export interface SendTransactionParams {
  to: string
  amount: bigint
}

export async function orchestrateSendTransaction(
  dependencies: SendTransactionDependencies,
  params: SendTransactionParams,
  logger: Logger
): Promise<TransactionReceipt>
```

### Orchestratorテストの方針

#### 1. 統合レベルのテスト分類

Orchestratorテストは**Khorikivの統合テスト**に分類されます：

- **スコープ**: `application-business-rules`層内のユースケース統合
- **モック対象**: `application-business-rules/interfaces`の抽象化（`Provider`, `Wallet`, `Sleeper`, `AddressValidator`）
- **実物使用**: managed dependencies（エンティティ、アダプター）

#### 2. テスト構造化パターン

```typescript
describe('orchestrateSendTransaction', () => {
  describe('successful workflow', () => {
    it('executes complete send workflow', async () => {})
    it('logs each step appropriately', async () => {})
  })

  describe('error handling at each step', () => {
    it('fails at balance check step', async () => {})
    it('fails at nonce retrieval step', async () => {})
    it('fails at fee retrieval step', async () => {})
    it('fails at transaction building step', async () => {})
    it('fails at signing step', async () => {})
    it('fails at broadcast step', async () => {})
    it('fails at confirmation step', async () => {})
  })

  describe('workflow validation', () => {
    it('calls steps in correct order', async () => {})
    it('uses managed dependencies correctly', async () => {})
  })

  describe('business rules enforcement', () => {
    it('enforces Sepolia chain ID', async () => {})
    it('enforces EIP1559 transaction type', async () => {})
  })
})
```

#### 3. 段階的エラーハンドリングテスト

各ワークフロー段階での失敗を個別にテストし、エラー伝播とステップ中断を検証：

```typescript
it('fails at balance check step', async () => {
  vi.mocked(mockProvider.getBalance).mockResolvedValue(BigInt('500000000000000000')) // Insufficient

  await expect(orchestrateSendTransaction(dependencies, params, mockLogger)).rejects.toThrow(
    'Insufficient funds'
  )

  // Should not proceed to further steps
  expect(mockProvider.getTransactionCount).not.toHaveBeenCalled()
  expect(mockProvider.getFeeData).not.toHaveBeenCalled()
})
```

#### 4. ワークフロー順序の検証

実行順序の正確性を保証するテスト：

```typescript
it('calls steps in correct order', async () => {
  const callOrder: string[] = []

  vi.mocked(mockProvider.getBalance).mockImplementation(async () => {
    callOrder.push('checkBalance')
    return BigInt('2000000000000000000')
  })

  // 他のステップも同様に実装...

  await orchestrateSendTransaction(dependencies, params, mockLogger)

  expect(callOrder).toEqual([
    'checkBalance',
    'getNonce',
    'getFees',
    'signTransaction',
    'sendSignedTransaction',
    'confirmTransaction',
  ])
})
```

#### 5. managed/unmanaged依存関係の検証

```typescript
it('uses managed dependencies correctly', async () => {
  await orchestrateSendTransaction(dependencies, params, mockLogger)

  // unmanaged dependencyのモック検証
  expect(mockAddressValidator.validate).toHaveBeenCalledWith(params.to)
  expect(mockProvider.getBalance).toHaveBeenCalledWith(mockWallet.address)

  // managed dependencyは実物使用（Transaction, adaptersなど）
  // 検証は最終的な出力やビジネスルールで行う
})
```

### Orchestratorパターンの利点

1. **完全なワークフローテスト**: 個別ユースケースだけでなく、それらの統合も検証
2. **エラー処理の網羅**: 各段階でのエラーハンドリングを体系的にテスト
3. **CLIの単純化**: エントリーポイントから複雑なロジックを排除
4. **保守性の向上**: ワークフロー変更時のテスト影響範囲を明確化
5. **デバッグの容易さ**: 各ステップの実行状況をログで追跡可能

### frameworks層の新しい役割

Orchestrator導入後の`frameworks/node/index.ts`：

```typescript
async function send(options: unknown): Promise<void> {
  // 1. バリデーション（CLI特有の責任）
  if (!isSendOptions(options)) {
    error('Invalid options for send command')
    return exit(1)
  }

  try {
    // 2. 依存関係の生成（Factory経由）
    const dependencies = createSendDependencies({
      privateKey: options.privateKey,
      infuraUrl: options.infuraUrl,
    })

    // 3. パラメータ変換（CLI特有の責任）
    const params = {
      to: options.to,
      amount: parseEtherInWei(options.amount),
    }

    // 4. ワークフロー実行（Orchestratorに委譲）
    await orchestrateSendTransaction(dependencies, params, { log })
    log('completed successfully')
  } catch (error: unknown) {
    return handleError('Send Error', error)
  }
}
```

**CLIの責任**:

- コマンドライン引数の解析と検証
- エラーメッセージの表示
- プロセス終了コードの設定
- 依存関係ファクトリーの呼び出し

**Orchestratorの責任**:

- ビジネスワークフローの実行
- ユースケース間の連携
- エラー伝播とログ出力
- ビジネスルールの強制

この分離により、CLIは薄いラッパーとなり、ビジネスロジックは完全にテスト可能なOrchestrator層で処理されます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oogatta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
