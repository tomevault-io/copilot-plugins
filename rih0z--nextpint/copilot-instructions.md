## nextpint

> Don't hold back. give it your all！

# Claude開発ガイドライン

## Ultrathink
Don't hold back. give it your all！
日本語で回答して。

## セキュリティチェックリスト

### GitHubへのプッシュ前に必ず確認

1. **APIキー・シークレットの確認**
   - [ ] `.env`ファイルが`.gitignore`に含まれているか
   - [ ] ハードコードされたAPIキーやパスワードがないか
   - [ ] データベースIDやトークンが公開されていないか

2. **個人情報の確認**
   - [ ] 個人のメールアドレスや電話番号が含まれていないか
   - [ ] 本名以外の個人識別情報がないか

3. **設定ファイルの確認**
   - [ ] `wrangler.toml`にシークレット情報が含まれていないか
   - [ ] データベースIDがプレースホルダーのままか

4. **URLの公開範囲**
   - [ ] READMEにはフロントエンドURL以外記載しない
   - [ ] APIエンドポイントURLは公開しない
   - [ ] 内部サービスのURLは記載しない

## 作業完了チェックリスト

### 作業完了時に実行

1. **GitHubへの追加**
   ```bash
   git add .
   git commit -m "適切なコミットメッセージ"
   git push origin main
   ```

2. **ビルドとデプロイ**
   ```bash
   # React Nativeアプリ（ローカル環境で実行）
   cd /path/to/NextPint
   npm install
   npm run ios  # または npm run android

   # Cloudflare Workers API
   cd api
   npm install
   npx wrangler login
   npx wrangler d1 create nextpint-db  # 初回のみ
   npm run db:init  # 初回のみ
   npm run db:seed  # 初回のみ
   npm run deploy
   ```

## デプロイ手順

### Cloudflare Workersのデプロイ

1. **初回セットアップ**
   - Cloudflareアカウントでログイン
   - D1データベースの作成
   - wrangler.tomlのdatabase_id更新

2. **デプロイコマンド**
   ```bash
   # Production
   npm run deploy

   # Development
   npx wrangler deploy --env development
   ```

3. **デプロイ後の確認**
   - ヘルスチェックエンドポイントの確認
   - テンプレートAPIの動作確認

## 本番環境のURL設定

### 固定URLの指定

1. **バックエンドAPI（Cloudflare Workers）**
   - 本番環境URL: `https://nextpint-api.YOUR-SUBDOMAIN.workers.dev`
   - このURLは固定で、変更しない
   - すべてのフロントエンドアプリケーションはこのURLを使用

2. **フロントエンドアプリケーション**
   - React Native: アプリ内でAPIのベースURLを上記に固定
   - 環境変数またはconfig.tsで管理
   ```typescript
   // src/config/api.ts
   export const API_BASE_URL = 'https://nextpint-api.YOUR-SUBDOMAIN.workers.dev';
   ```

3. **通信の確実性を保証するための設定**
   - CORSを適切に設定（wrangler.toml内）
   - APIのヘルスチェックエンドポイントを定期的に確認
   - ネットワークエラーハンドリングの実装

### デプロイ時の確認事項

1. **API URLの固定化**
   - [ ] フロントエンドのAPI URLが本番環境URLに設定されているか
   - [ ] 開発環境と本番環境のURLが混在していないか
   - [ ] APIのベースURLがハードコードされず、設定ファイルで管理されているか

2. **通信の検証**
   - [ ] デプロイ後、フロントエンドからAPIへの接続テスト実施
   - [ ] CORSエラーが発生していないか確認
   - [ ] APIレスポンスが正常に返されるか確認

## デザインガイドライン

### アイコンの使用

1. **絵文字の使用禁止**
   - 一切絵文字を使用しない（UI、コメント、テキスト全て）
   - 代わりにreact-iconsライブラリを使用

2. **react-iconsの使用**
   ```bash
   npm install react-icons
   ```
   
   ```typescript
   // 推奨アイコンライブラリ
   import { FaHome, FaBeer, FaUser } from 'react-icons/fa';
   import { IoMdSettings, IoMdAdd } from 'react-icons/io';
   import { MdAnalytics, MdHistory } from 'react-icons/md';
   import { BiImport } from 'react-icons/bi';
   ```

3. **アイコンの統一性**
   - 同じ機能には必ず同じアイコンを使用
   - サイズとスタイルを統一（16px, 24px, 32px等）
   - カラーはテーマに合わせて設定

### UIコンポーネントのガイドライン

1. **アイコンボタンの実装**
   ```typescript
   <Button>
     <FaHome size={20} />
     ホーム
   </Button>
   ```

2. **ナビゲーションでのアイコン使用**
   - タブナビゲーション: アイコン + ラベル
   - メニュー項目: アイコン + テキスト

## 実装品質基準

### SOLID原則の遵守

1. **単一責任の原則 (Single Responsibility Principle)**
   - 各クラス・モジュールは単一の責任のみを持つ
   - 例: PromptServiceはプロンプト関連、StorageServiceはストレージ関連のみ
   - 変更理由は常に1つに限定される

2. **オープン・クローズドの原則 (Open/Closed Principle)**
   - 拡張に対して開いていて、修正に対して閉じている
   - 新機能追加時は既存コードを変更せず、新しいクラスやメソッドを追加
   - インターフェースや抽象クラスを活用

3. **リスコフの置換原則 (Liskov Substitution Principle)**
   - 派生クラスは基底クラスと置換可能でなければならない
   - インターフェースの実装は完全に契約を満たす
   - 例: IPromptService、IStorageServiceの全実装は互換性を持つ

4. **インターフェース分離の原則 (Interface Segregation Principle)**
   - クライアントが必要としないメソッドへの依存を強制しない
   - 大きなインターフェースより小さな特化したインターフェースを推奨
   - 例: IStorageServiceを細分化（IBeerStorage、ISessionStorage等）

5. **依存性逆転の原則 (Dependency Inversion Principle)**
   - 高レベルモジュールは低レベルモジュールに依存してはならない
   - 両者とも抽象に依存すべき
   - 例: ServiceFactoryパターンで具象クラスではなくインターフェースに依存

### コード品質チェックリスト

1. **アーキテクチャ設計**
   - [ ] 各サービスが単一の責任を持っているか
   - [ ] 依存関係が適切に注入されているか
   - [ ] インターフェースと実装が分離されているか
   - [ ] 循環依存が存在しないか

2. **実装の品質**
   - [ ] DRY原則（Don't Repeat Yourself）に従っているか
   - [ ] エラーハンドリングが適切か
   - [ ] 型安全性が保証されているか（TypeScript）
   - [ ] 適切な抽象化レベルか

3. **テスタビリティ**
   - [ ] 単体テストが容易に書けるか
   - [ ] モックやスタブが簡単に作成できるか
   - [ ] テストカバレッジが十分か（目標: 80%以上）
   - [ ] 統合テストが実施可能か

### 実装例：ServiceFactoryパターン

```typescript
// 良い例：SOLID原則に従った実装
interface IService {
  performAction(): void;
}

class ServiceFactory {
  private static services: Map<string, IService> = new Map();
  
  static register(name: string, service: IService): void {
    this.services.set(name, service);
  }
  
  static get<T extends IService>(name: string): T {
    return this.services.get(name) as T;
  }
}

// 悪い例：SOLID原則に反する実装
class BadService {
  // 複数の責任を持つ
  saveData() { /* ... */ }
  sendEmail() { /* ... */ }
  generateReport() { /* ... */ }
  
  // 具象クラスに直接依存
  constructor(private db: PostgresDB) {}
}
```

### リファクタリング指針

1. **単一責任違反を発見したら**
   - クラスを複数に分割
   - 各クラスに明確な責任を割り当て
   - ServiceFactoryで依存関係を管理

2. **インターフェース違反を発見したら**
   - 大きなインターフェースを小さく分割
   - 必要なメソッドのみを含むインターフェースを作成
   - 実装クラスで必要なインターフェースのみ実装

3. **依存性の問題を発見したら**
   - 依存性注入（DI）パターンを使用
   - ServiceFactoryを活用
   - インターフェースへの依存に変更

## 重要な注意事項

- **絶対にシークレット情報をコミットしない**
- **database_idは必ずプレースホルダーのままにする**
- **個人情報を含むファイルは作成しない**
- **デプロイ前に必ずローカルでテストする**
- **本番環境のAPIのURLは必ず固定のものを使用する**
- **フロントエンドとバックエンドの通信は必ず本番環境の固定URLで行う**
- **絵文字は一切使用せず、react-iconsを使用する**
- **SOLID原則を常に意識し、コードの品質を維持する**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rih0z) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
