## gftd-densha

> Densha Framework Naming Conventions - 命名規則ガイドライン


# Densha Framework 命名規則

このドキュメントは、Denshaフレームワーク全体で使用する命名規則を定義します。コードの可読性、一貫性、保守性を高めるためのガイドラインです。

## 1. 基本原則

### 1.1 一貫性 (Consistency)
- **英語で統一する**: すべての識別子は英語を使用
- **統一された用語を使用**: 同じ概念には同じ名前を使用
- **省略形を避ける**: 明確な単語を使用

### 1.2 明確性 (Clarity)
- **目的が伝わる名前**: 何をするものか一目でわかる
- **文脈を考慮**: スコープ内で一意性を確保
- **一般用語を優先**: ドメイン特化しすぎない

### 1.3 簡潔性 (Brevity)
- **冗長な接頭辞/接尾辞を避ける**: `create`, `get`, `set`などの多用を避ける
- **長すぎる名前を避ける**: 3-4単語以内に収める
- **意味のある短縮形を使用**: 広く認知された略語のみ使用

## 2. 型定義の命名規則

### 2.1 基本型 (Base Types)
```typescript
// ✅ 良い例
type User = { ... }
type Order = { ... }
type Product = { ... }

// ❌ 避ける例
type UserEntity = { ... }
type OrderData = { ... }
type ProductModel = { ... }
```

### 2.2 複合型 (Compound Types)
```typescript
// ✅ 良い例
type UserWithProfile = { ... }
type OrderStatus = 'pending' | 'confirmed' | 'shipped'
type ApiResponse<T> = { data: T; error?: string }

// ❌ 避ける例
type UserWithProfileData = { ... }
type OrderStatusType = 'pending' | 'confirmed' | 'shipped'
type ApiResponseResult<T> = { data: T; error?: string }
```

### 2.3 関数型 (Function Types)
```typescript
// ✅ 良い例
type Handler<T> = (input: T) => void
type Validator<T> = (value: T) => boolean
type Transform<I, O> = (input: I) => O

// ❌ 避ける例
type HandlerFunction<T> = (input: T) => void
type ValidationFunction<T> = (value: T) => boolean
type TransformationFunction<I, O> = (input: I) => O
```

## 3. 関数・メソッドの命名規則

### 3.1 基本関数 (Core Functions)
```typescript
// ✅ 良い例
const user = (config: UserConfig) => { ... }
const order = (config: OrderConfig) => { ... }
const product = (config: ProductConfig) => { ... }

// ❌ 避ける例
const createUser = (config: UserConfig) => { ... }
const makeOrder = (config: OrderConfig) => { ... }
const newProduct = (config: ProductConfig) => { ... }
```

### 3.2 ヘルパー関数 (Helper Functions)
```typescript
// ✅ 良い例
const attr = (schema: Schema, defaultValue?) => { ... }
const action = (config: ActionConfig) => { ... }
const policy = (fn: PolicyFn) => { ... }

// ❌ 避ける例
const createAttribute = (schema: Schema, defaultValue?) => { ... }
const defineAction = (config: ActionConfig) => { ... }
const createPolicy = (fn: PolicyFn) => { ... }
```

### 3.3 クエリ・操作関数 (Query/Operation Functions)
```typescript
// ✅ 良い例
const find = (query: Query) => { ... }
const filter = (predicate: Predicate) => { ... }
const sort = (comparator: Comparator) => { ... }

// ❌ 避ける例
const findByQuery = (query: Query) => { ... }
const filterWithPredicate = (predicate: Predicate) => { ... }
const sortByComparator = (comparator: Comparator) => { ... }
```

## 4. 変数・定数の命名規則

### 4.1 変数 (Variables)
```typescript
// ✅ 良い例
const user = createUser(config)
const orders = await fetchOrders(userId)
const isValid = validateEmail(email)

// ❌ 避ける例
const userEntity = createUser(config)
const orderList = await fetchOrders(userId)
const isEmailValid = validateEmail(email)
```

### 4.2 定数 (Constants)
```typescript
// ✅ 良い例
const MAX_RETRY_COUNT = 3
const API_TIMEOUT = 5000
const CACHE_TTL = 3600000

// ❌ 避ける例
const MAXIMUM_RETRY_COUNT = 3
const API_TIMEOUT_IN_MS = 5000
const CACHE_TIME_TO_LIVE = 3600000
```

## 5. クラス・インターフェースの命名規則

### 5.1 インターフェース (Interfaces)
```typescript
// ✅ 良い例
interface Handler<T> { ... }
interface Repository<T> { ... }
interface Service { ... }

// ❌ 避ける例
interface IHandler<T> { ... }
interface IRepository<T> { ... }
interface IService { ... }
```

### 5.2 クラス (Classes)
```typescript
// ✅ 良い例
class UserService { ... }
class OrderRepository { ... }
class ProductHandler { ... }

// ❌ 避ける例
class UserServiceImpl { ... }
class OrderDataAccessObject { ... }
class ProductHandlerManager { ... }
```

## 6. ファイルの命名規則

### 6.1 ファイル名 (File Names)
```typescript
// ✅ 良い例
user.ts
order.ts
product.ts
user-service.ts
order-repository.ts

// ❌ 避ける例
user-entity.ts
order-model.ts
product-data.ts
user-service-impl.ts
order-repository-dao.ts
```

### 6.2 ディレクトリ名 (Directory Names)
```typescript
// ✅ 良い例
domain/
  user.ts
  order.ts
  product.ts

infrastructure/
  user-repository.ts
  order-repository.ts

application/
  user-service.ts
  order-service.ts

// ❌ 避ける例
domain/
  user-entity.ts
  order-model.ts

infrastructure/
  user-repository-impl.ts
  order-repository-dao.ts

application/
  user-service-manager.ts
  order-service-handler.ts
```

## 7. 命名パターン一覧

### 7.1 許可されるパターン
```typescript
// 基本パターン
PascalCase: 型定義、クラス、インターフェース
camelCase: 関数、変数、メソッド、プロパティ
SCREAMING_SNAKE_CASE: 定数
kebab-case: ファイル名、ディレクトリ名

// 具体例
type User = { ... }                    // PascalCase
const user = (config) => { ... }       // camelCase
const MAX_RETRIES = 3                  // SCREAMING_SNAKE_CASE
user-service.ts                        // kebab-case
```

### 7.2 避けるべきパターン
```typescript
// ❌ 避けるパターン
type UserEntity = { ... }              // Entityサフィックス
const createUser = () => { ... }       // createプレフィックス
const userData = { ... }               // Dataサフィックス
interface IUser { ... }                // Iプレフィックス
class UserServiceImpl { ... }          // Implサフィックス
```

## 8. Densha Framework アーキテクチャ - Ash Framework完全対応

### 8.1 Ash Framework API対応表

DenshaはAsh FrameworkのAPI構造を可能な限り忠実に再現し、Effectによる高度な型安全とWASM Component Modelを統合しています。

#### 🎯 **Resources (主要概念)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Resource** | `Resource` | ✅ 完全対応 | AshのResource DSLを再現 |
| **Ash.Resource.Calculation** | `calculations` | ✅ 完全対応 | Calculation DSL |
| **Ash.Resource.ManualCreate** | `manualCreate` | ✅ **完全対応** | 承認ワークフロー、詳細設定付き |
| **Ash.Resource.ManualUpdate** | `manualUpdate` | ✅ **完全対応** | 楽観的ロック、承認機能付き |
| **Ash.Resource.ManualRead** | `manualRead` | ✅ **完全対応** | フィールド選択、リレーション読み込み |
| **Ash.Resource.ManualDestroy** | `manualDestroy` | ✅ **完全対応** | ソフト削除、承認機能付き |
| **Ash.Resource.ManualRelationship** | `manualRelationship` | ✅ **完全対応** | 手動リレーションシップ処理 |
| **Ash.Resource.Preparation** | `preparations` | ✅ 完全対応 | Preparation DSL |
| **Ash.Resource.Change** | `changes` | ✅ 完全対応 | Change DSL |
| **Ash.Resource.Validation** | `validations` | ✅ 完全対応 | Validation DSL |

#### 🔍 **Queries (クエリ)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Query** | `Query` | ✅ **完全対応** | 完全なクエリビルダーシステム |
| **Ash.Query.Aggregate** | `aggregates` | ✅ **完全対応** | Aggregate DSL |
| **Ash.Query.Calculation** | `calculations` | ✅ **完全対応** | Calculation in Query |

#### 🔄 **Changes (変更)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Resource.Change.Builtins** | `changeBuiltins` | ✅ **完全対応** | すべてのビルトイン変更機能 |
| **Ash.Changeset** | `Changeset` | ✅ **完全対応** | Changeset DSL |
| **Ash.Changeset.ManagedRelationshipHelpers** | `managedRelationships` | 🔄 **部分対応** | 管理リレーションシップ |

#### ✅ **Validations (検証)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Resource.Validation.Builtins** | `validationBuiltins` | ✅ **完全対応** | すべてのビルトイン検証機能 |
| **Ash.Resource.Validation** | `validations` | ✅ **完全対応** | Validation DSL |

#### 🔐 **Authorization (認可)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Authorizer** | `Authorizer` | 🔄 **部分対応** | オーソライザー |
| **Ash.Policy.Check** | `Policy` | ✅ **完全対応** | Policy DSL |
| **Ash.Policy.Check.Builtins** | `policyBuiltins` | ✅ **完全対応** | すべてのビルトインポリシー |
| **Ash.Policy.FilterCheck** | `FilterCheck` | ✅ **完全対応** | フィルターチェック |
| **Ash.Policy.SimpleCheck** | `SimpleCheck` | ✅ **完全対応** | シンプルチェック |

#### 📦 **Extensions (拡張)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.DataLayer.* ** | `dataLayer` | ✅ **完全対応** | Memory, PostgreSQL, ETSアダプター |
| **Ash.Notifier.PubSub** | `notifier` | ✅ **完全対応** | PubSub, Email, SMS, Webhook |
| **Ash.Policy.Authorizer** | `authorizer` | ✅ **完全対応** | Simple, Policy, Customオーソライザー |
| **Ash.CodeInterface.REST** | `codeInterface.rest` | ✅ **完全対応** | REST APIコード生成 |
| **Ash.CodeInterface.GraphQL** | `codeInterface.graphql` | ✅ **完全対応** | GraphQL APIコード生成 |
| **Ash.CodeInterface.JSON:API** | `codeInterface.jsonApi` | ✅ **完全対応** | JSON:APIコード生成 |
| **Ash.Domain** | `domain` | ✅ **完全対応** | ドメイン管理機能 |
| **Ash.Domain.Info** | `domainInfo` | ✅ **完全対応** | ドメイン情報取得 |
| **Ash.Reactor** | `reactor` | ✅ **完全対応** | Event, Stream, Stateリアクター |
| **Ash.Test** | `test` | ✅ **完全対応** | テストスイート、テストケース |
| **Ash.Generator** | `generator` | ✅ **完全対応** | テストデータ生成 |
| **Ash.Seed** | `seed` | ✅ **完全対応** | シードデータ管理 |
| **Ash.BulkResult** | `utilities.bulkResult` | ✅ **完全対応** | バルク操作結果 |
| **Ash.CiString** | `utilities.ciString` | ✅ **完全対応** | 大文字小文字無視文字列 |
| **Ash.Expr** | `utilities.expr` | ✅ **完全対応** | 式評価 |
| **Ash.Filter** | `utilities.filter` | ✅ **完全対応** | 高度なフィルター |
| **Ash.Page** | `utilities.page` | ✅ **完全対応** | ページネーション |
| **Ash.Sort** | `utilities.sort` | ✅ **完全対応** | 高度なソート |
| **Ash.UUID** | `utilities.uuid` | ✅ **完全対応** | UUID生成・操作 |
| **Ash.Resource** | `Resource` | ✅ **完全対応** | リソース拡張 |

#### 🔍 **Introspection (イントロスペクション)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.*.Info** | `Info` | 🔄 部分対応 | 情報取得 |
| **Ash.Domain.Info** | `DomainInfo` | 🔄 部分対応 | ドメイン情報 |
| **Ash.Resource.Info** | `ResourceInfo` | 🔄 部分対応 | リソース情報 |

#### 📊 **Testing (テスト)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Generator** | `Generator` | 🔄 部分対応 | ジェネレーター |
| **Ash.Seed** | `Seed` | 🔄 部分対応 | シード |
| **Ash.Test** | `Test` | 🔄 部分対応 | テスト |

#### 🧩 **Builtins (ビルトイン)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Policy.Check.Builtins** | `policyCheckBuiltins` | ✅ **完全対応** | すべてのポリシービルトイン |
| **Ash.Resource.Change.Builtins** | `changeBuiltins` | ✅ **完全対応** | すべての変更ビルトイン |
| **Ash.Resource.Validation.Builtins** | `validationBuiltins` | ✅ **完全対応** | すべての検証ビルトイン |

#### 🎨 **Visualizations (可視化)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Domain.Info.Diagram** | `DomainDiagram` | 🔄 部分対応 | ドメイン図 |
| **Ash.Policy.Chart.Mermaid** | `PolicyChart` | 🔄 部分対応 | ポリシーチャート |

#### 🔄 **Tracing (トレーシング)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Tracer** | `Tracer` | 🔄 部分対応 | トレーサー |
| **Ash.Tracer.Simple** | `SimpleTracer` | 🔄 部分対応 | シンプルトレーサー |
| **Ash.Tracer.Simple.Span** | `Span` | 🔄 部分対応 | スパン |

#### 🛠️ **Utilities (ユーティリティ)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.BulkResult** | `BulkResult` | 🔄 部分対応 | バルク結果 |
| **Ash.Changeset.ManagedRelationshipHelpers** | `ManagedRelationshipHelpers` | 🔄 部分対応 | 管理リレーションシップ |
| **Ash.CiString** | `CiString` | 🔄 部分対応 | 大文字小文字無視文字列 |
| **Ash.Expr** | `Expr` | 🔄 部分対応 | 式 |
| **Ash.Filter** | `Filter` | 🔄 部分対応 | フィルター |
| **Ash.Page** | `Page` | 🔄 部分対応 | ページネーション |
| **Ash.Sort** | `Sort` | 🔄 部分対応 | ソート |
| **Ash.UUID** | `UUID` | 🔄 部分対応 | UUID |

#### 📝 **Types (型システム)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|
| **Ash.Type.Atom** | `Atom` | ✅ **完全対応** | アトム型（制約付き） |
| **Ash.Type.String** | `String` | ✅ **完全対応** | 文字列型（制約付き） |
| **Ash.Type.Integer** | `Integer` | ✅ **完全対応** | 整数型（制約付き） |
| **Ash.Type.Float** | `Float` | ✅ **完全対応** | 浮動小数点型（制約付き） |
| **Ash.Type.Boolean** | `Boolean` | ✅ **完全対応** | 真偽値型（制約付き） |
| **Ash.Type.Date** | `Date` | ✅ **完全対応** | 日付型（制約付き） |
| **Ash.Type.UUID** | `UUID` | ✅ **完全対応** | UUID型（制約付き） |
| **Ash.Type.Map** | `Map` | ✅ **完全対応** | マップ型 |
| **Ash.Type.Struct** | `Struct` | ✅ **完全対応** | 構造体型 |
| **Ash.Type.Email** | `Email` | ✅ **完全対応** | メールアドレス型 |
| **Ash.Type.Url** | `Url` | ✅ **完全対応** | URL型 |
| **Ash.Type.Decimal** | `Decimal` | ✅ **完全対応** | Decimal型 |
| **Ash.Type.CiString** | `CiString` | ✅ **完全対応** | 大文字小文字無視文字列 |
| **Ash.Type.Vector** | `Vector` | ✅ **完全対応** | ベクター型 |
| **Ash.Type.Tuple** | `Tuple` | ✅ **完全対応** | タプル型 |

### 8.2 アーキテクチャ階層
DenshaはActorを最上位概念として位置づけ、WASM Component Modelによるコンポーネント化をサポートします。

```
┌─────────────────────────────────────────────────────────────┐
│                        Actor                                │  ← Denshaプロジェクト全体
│  (Densha Project - WASM Component + WPRC/WIT)              │
├─────────────────────────────────────────────────────────────┤
│  • Service間通信                                            │
│  • コンポーネントライフサイクル                              │
│  • プロジェクト統合                                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   ResourceProcess                           │  ← 個別リソースの処理
│  (Effect Fiberベース - Resource実行単位)                    │
├─────────────────────────────────────────────────────────────┤
│  • UserResource, PostResourceなどの処理                     │
│  • Effect Fiberによる軽量実行                              │
│  • 内部状態管理                                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┤
│                        Resource                             │  ← ドメインリソース
│  (Ash Framework準拠 - 宣言的定義)                          │
├─────────────────────────────────────────────────────────────┤
│  • Resource/Action/Relationships定義                       │
│  • Schema + Validations分離                                │
│  • ビジネスロジック                                        │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Actor概念の上流化
#### 🎭 **Actor (最上位)**
```typescript
// Denshaプロジェクト全体を表すActor
const DenshaActor = {
  name: "my-densha-app",
  version: "1.0.0",

  // WASM Componentとしての機能
  component: {
    wit: "interface.densha",     // WITファイル
    wprc: "densha.wprc",         // WPRC定義
    wasm: "densha.wasm",         // コンパイル済みWASM
  },

  // サービス間通信
  services: {
    userService: UserActor,
    postService: PostActor,
    notificationService: NotificationActor,
  },

  // コンポーネント統合
  compose: (services) => ({
    userManagement: services.userService,
    contentManagement: services.postService,
    notifications: services.notificationService,
  }),
}
```

#### 🔄 **ResourceProcess (実行層)**
```typescript
// 個別リソースの処理単位
const UserResourceProcess = {
  resource: UserResource,

  // Effect Fiberによる実行
  fiber: Effect.fiber(() => {
    // ResourceのActionを実行
    return userResource.actions.create.run({
      input: createUserInput,
      context: actionContext,
    })
  }),

  // 状態管理
  state: {
    currentUser: Ref.make(null),
    userSessions: Map<string, UserSession>,
  },
}
```

### 8.3 WASM Component Model統合
#### 🧩 **WIT (WebAssembly Interface Types)**
```wit
// densha.wit - Densha Component Interface
interface densha {
  // Resource operations
  resource user {
    constructor(email: string, name: string)
    create: func(input: create-input) -> user
    read: func(id: string) -> option<user>
    update: func(id: string, input: update-input) -> user
    delete: func(id: string) -> unit
  }

  // Actor operations
  actor: func(name: string) -> actor
  send-message: func(to: actor, message: actor-message) -> unit
  call-actor: func(to: actor, message: actor-message) -> result<any, error>

  // Types
  type create-input = {
    email: string,
    name: string,
    role: string,
  }

  type update-input = {
    email: option<string>,
    name: option<string>,
    role: option<string>,
  }

  type actor-message = {
    resource: string,
    action: string,
    payload: any,
  }
}
```

#### 🔗 **WPRC (WebAssembly Procedure Call)**
```wprc
// densha.wprc - Component間の通信プロトコル
component user-service {
  import notification-service: "notification.wit"
  import database: "database.wit"

  export user-management: interface {
    create-user: func(request: create-user-request) -> result<user, error>
    get-user: func(id: string) -> option<user>
    update-user: func(id: string, updates: user-updates) -> result<user, error>
    delete-user: func(id: string) -> result<unit, error>
  }
}

component notification-service {
  export notification: interface {
    send-welcome-email: func(to: string, user: user) -> result<unit, error>
    send-notification: func(to: string, template: string, data: any) -> result<unit, error>
  }
}
```

### 8.4 実装レイヤー
#### 🌐 **Component Layer (WASM)**
```typescript
// WASM Componentとして実行
@component("densha")
export class DenshaComponent {
  @export
  async createUser(request: CreateUserRequest): Promise<User> {
    // WIT経由でResourceProcessを呼び出し
    return await this.userResourceProcess.create(request)
  }

  @export
  async sendMessage(to: string, message: ActorMessage): Promise<void> {
    // WPRC経由で他のComponentにメッセージ送信
    await this.wprc.send(to, message)
  }
}
```

#### ⚡ **Runtime Layer (Effect)**
```typescript
// Effectランタイムでの実行
const denshaRuntime = new DenshaRuntime({
  // WASM Componentの読み込み
  components: {
    "user-service": loadWasmComponent("./user-service.wasm"),
    "post-service": loadWasmComponent("./post-service.wasm"),
  },

  // Actor間の通信
  actors: {
    userActor: createActor("user", UserResource),
    postActor: createActor("post", PostResource),
  },

  // Effect Fiberによる並行処理
  fibers: {
    userFiber: Effect.fiber(() => userActor.run()),
    postFiber: Effect.fiber(() => postActor.run()),
  },
})
```

### 8.5 コンポーネント化の利点
#### 🔄 **疎結合アーキテクチャ**
```typescript
// 各Componentは独立してデプロイ可能
const userComponent = createWasmComponent({
  wit: "user.wit",
  wprc: "user.wprc",
  wasm: compileToWasm(UserService),
})

const postComponent = createWasmComponent({
  wit: "post.wit",
  wprc: "post.wprc",
  wasm: compileToWasm(PostService),
})

// ランタイムでの動的統合
const app = composeComponents({
  user: userComponent,
  post: postComponent,
  notification: notificationComponent,
})
```

#### 📦 **配布と再利用**
```typescript
// WASMバイナリとして配布
const denshaPackage = {
  name: "densha-crm-app",
  version: "1.0.0",
  components: {
    "user-management": "user-service.wasm",
    "content-management": "post-service.wasm",
    "notification": "notification-service.wasm",
  },
  wit: "densha.wit",
  wprc: "densha.wprc",
}

// 他のプロジェクトでの再利用
const myApp = useDenshaPackage(denshaPackage, {
  config: { databaseUrl: "..." },
  customize: {
    userService: { roles: ["admin", "user"] },
  },
})
```

このアーキテクチャにより、Denshaは単なるフレームワークではなく、コンポーネントベースの分散システムプラットフォームとなります。

## 9. 適用ガイドライン

### 9.1 優先順位
1. **可読性** > 簡潔性 > 慣習
2. **明確性** > 完全性 > 簡略化
3. **一貫性** > 個別最適化 > 個人的好み

### 9.2 例外の扱い
- 広く使われている外部APIとの連携時は、外部の命名規則に従う
- フレームワークのコア概念はフレームワークの規則を優先
- チームの合意があれば、プロジェクト固有の調整を許可

### 9.3 移行戦略
1. 新規コードから新しい命名規則を適用
2. 既存コードは段階的にリファクタリング
3. 破壊的な変更はメジャーバージョンアップデート時に実施

## 10. 参考・関連資料

- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Microsoft TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

*この命名規則は、Denshaフレームワークの進化に伴い更新されます。最新版は常にこのドキュメントを参照してください。*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gftdcojp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
