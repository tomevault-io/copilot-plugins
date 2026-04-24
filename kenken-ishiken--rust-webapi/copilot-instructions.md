## rust-webapi

> このドキュメントは、Phase 3で実装されたエラーハンドリングとメトリクス記録の統一パターンを説明します。一貫性のあるエラー処理と観測性の向上を実現するためのベストプラクティスを提供します。

# エラーハンドリング・メトリクス統一ガイド

このドキュメントは、Phase 3で実装されたエラーハンドリングとメトリクス記録の統一パターンを説明します。一貫性のあるエラー処理と観測性の向上を実現するためのベストプラクティスを提供します。

## 🎯 目的

- **エラーハンドリングの統一**: `AppError`による一貫したエラー処理
- **メトリクス記録の標準化**: 統一されたメトリクスAPIの使用
- **観測性の向上**: tracingとの統合による詳細なログ記録
- **保守性の向上**: 重複コードの削減と可読性の向上

## 🚨 エラーハンドリング統一

### AppError の使用

すべてのエラーは [src/infrastructure/error.rs](mdc:src/infrastructure/error.rs) で定義された `AppError` を使用します。

#### ✅ 推奨パターン

```rust
use crate::infrastructure::error::{AppError, AppResult};

// 基本的なエラー処理
pub async fn find_by_id(&self, id: u64) -> AppResult<ItemResponse> {
    let item = self.repository.find_by_id(id).await?;
    match item {
        Some(item) => Ok(self.to_response(item)),
        None => Err(AppError::not_found("Item", id)),
    }
}

// ヘルパーメソッドの使用
AppError::not_found("User", user_id)
AppError::validation_error("Invalid email format")
AppError::internal_error("Database connection failed")
```

#### ❌ アンチパターン

```rust
// unwrap/expect の使用（本番コードでは禁止）
let item = repository.find_by_id(id).await.unwrap();

// 直接的なエラー構築（ヘルパーメソッドを使用すべき）
Err(AppError::NotFound(format!("Item with id {} not found", id)))

// anyhow::Error の直接使用（AppErrorに変換すべき）
return Err(anyhow::anyhow!("Something went wrong"));
```

### エラーレスポンスの標準化

APIエラーレスポンスは以下の形式で統一されています：

```json
{
  "type": "NotFound",
  "message": "Item with id 123 not found",
  "timestamp": "2024-12-01T12:00:00Z"
}
```

## 📊 メトリクス統一

### 統一メトリクスAPI

[src/infrastructure/metrics/mod.rs](mdc:src/infrastructure/metrics/mod.rs) で定義されたメトリクスAPIを使用します。

#### ✅ 推奨パターン

```rust
use crate::infrastructure::metrics::Metrics;

// Result型の自動成功/失敗記録
pub async fn find_by_id(&self, id: u64) -> AppResult<ItemResponse> {
    Metrics::with_metrics("item", "find_by_id", async {
        let item = self.repository.find_by_id(id).await?;
        match item {
            Some(item) => Ok(self.to_response(item)),
            None => Err(AppError::not_found("Item", id)),
        }
    }).await
}

// 自動時間測定
pub async fn create(&self, req: CreateRequest) -> AppResult<Response> {
    Metrics::with_timer("service", "create", async {
        // 処理...
        result
    }).await
}

// 個別記録
Metrics::record_success("service", "operation");
Metrics::record_error("service", "operation");
```

#### MetricsTimer の使用

```rust
use crate::infrastructure::metrics::MetricsTimer;

pub async fn complex_operation(&self) -> AppResult<()> {
    let timer = MetricsTimer::new("service", "complex_operation");
    
    // 複雑な処理...
    
    timer.observe(); // 手動で記録
    // または Drop時に自動記録
    Ok(())
}
```

#### ❌ アンチパターン

```rust
// 古いメトリクスAPIの直接使用（統一APIを使用すべき）
increment_success_counter("service", "operation");
increment_error_counter("service", "operation");

// メトリクス記録の漏れ
pub async fn operation(&self) -> AppResult<()> {
    // メトリクス記録なし
    self.repository.do_something().await
}
```

## 🔍 実装例

### サービス層での統一パターン

[src/application/service/item_service.rs](mdc:src/application/service/item_service.rs) の実装例：

```rust
impl ItemService {
    pub async fn find_all(&self) -> AppResult<Vec<ItemResponse>> {
        Metrics::with_metrics("item", "find_all", async {
            let items = self.repository.find_all().await?;
            Ok(items
                .into_iter()
                .map(|item| self.to_response(item))
                .collect())
        }).await
    }

    pub async fn create(&self, req: CreateItemRequest) -> AppResult<ItemResponse> {
        Metrics::with_metrics("item", "create", async {
            // バリデーション
            self.validate_create_request(&req)?;
            
            // エンティティ作成
            let item = Item::new(req.name, req.description);
            
            // リポジトリ呼び出し
            let created_item = self.repository.create(item).await?;
            
            Ok(self.to_response(created_item))
        }).await
    }
}
```

### バッチ処理での個別メトリクス記録

```rust
pub async fn batch_delete(&self, req: BatchDeleteRequest) -> AppResult<BatchDeleteResponse> {
    let timer = MetricsTimer::new("item", "batch_delete");
    
    let successful_ids = self.repository.batch_delete(req.ids, req.is_physical).await?;
    let failed_ids: Vec<u64> = req.ids
        .into_iter()
        .filter(|id| !successful_ids.contains(id))
        .collect();

    // 個別に成功/失敗をメトリクスに記録
    if !successful_ids.is_empty() {
        Metrics::record_success("item", "batch_delete");
    }
    if !failed_ids.is_empty() {
        Metrics::record_error("item", "batch_delete");
    }

    timer.observe();
    Ok(BatchDeleteResponse { successful_ids, failed_ids })
}
```

## 📝 ログ統合

### tracing との統合

メトリクスAPIは自動的にtracingログを出力します：

```rust
// 自動的に以下のログが出力される
// DEBUG: Metrics: item find_by_id success
// DEBUG: Metrics: item find_by_id duration: 0.025s
// WARN:  Metrics: item find_by_id error
```

### 構造化ログの活用

```rust
use tracing::{info, warn, error};

pub async fn operation(&self, id: u64) -> AppResult<()> {
    info!(operation = "start", entity_id = id, "Starting operation");
    
    let result = Metrics::with_metrics("service", "operation", async {
        // 処理...
        Ok(())
    }).await;
    
    match &result {
        Ok(_) => info!(operation = "complete", entity_id = id, "Operation completed"),
        Err(e) => error!(operation = "failed", entity_id = id, error = %e, "Operation failed"),
    }
    
    result
}
```

## 🧪 テストでのエラーハンドリング

### テストコードでのパターン

```rust
#[tokio::test]
async fn test_find_by_id_not_found() {
    let mut mock_repo = MockItemRepository::new();
    mock_repo
        .expect_find_by_id()
        .with(eq(999u64))
        .return_once(|_| Ok(None));

    let service = ItemService::new(Arc::new(mock_repo));
    let result = service.find_by_id(999).await;

    // AppErrorの特定の型をテスト
    assert!(matches!(result, Err(AppError::NotFound(_))));
}
```

## 🔧 移行ガイド

### 既存コードの移行手順

1. **エラー型の統一**
   ```rust
   // Before
   return Err(anyhow::anyhow!("Not found"));
   
   // After
   return Err(AppError::not_found("Entity", id));
   ```

2. **メトリクス記録の統一**
   ```rust
   // Before
   increment_success_counter("service", "operation");
   
   // After
   Metrics::with_metrics("service", "operation", async {
       // 処理...
   }).await
   ```

3. **unwrap/expect の除去**
   ```rust
   // Before
   let item = repository.find_by_id(id).await.unwrap();
   
   // After
   let item = repository.find_by_id(id).await?;
   ```

## 📊 メトリクス出力形式

### Prometheus形式

```
# HELP api_success_total Total number of successful API requests
# TYPE api_success_total counter
api_success_total{service="item",endpoint="find_by_id"} 42

# HELP api_error_total Total number of API errors
# TYPE api_error_total counter
api_error_total{service="item",endpoint="find_by_id"} 3

# HELP api_request_duration_seconds API request duration in seconds
# TYPE api_request_duration_seconds histogram
api_request_duration_seconds_bucket{service="item",endpoint="find_by_id",le="0.005"} 10
api_request_duration_seconds_bucket{service="item",endpoint="find_by_id",le="0.01"} 25
api_request_duration_seconds_sum{service="item",endpoint="find_by_id"} 1.234
api_request_duration_seconds_count{service="item",endpoint="find_by_id"} 45
```

## 🎯 ベストプラクティス

### Do's

- ✅ `AppError`のヘルパーメソッドを使用
- ✅ `Metrics::with_metrics`で自動記録
- ✅ `MetricsTimer`で時間測定
- ✅ 構造化ログでコンテキスト情報を記録
- ✅ テストでエラー型を明示的に検証

### Don'ts

- ❌ 本番コードで`unwrap`/`expect`を使用
- ❌ 古いメトリクスAPIを直接使用
- ❌ エラーメッセージを直接構築
- ❌ メトリクス記録を忘れる
- ❌ ログレベルを適切に設定しない

## 🔗 関連ファイル

- **[src/infrastructure/error.rs](mdc:src/infrastructure/error.rs)** - AppError定義
- **[src/infrastructure/metrics/mod.rs](mdc:src/infrastructure/metrics/mod.rs)** - メトリクスAPI
- **[src/application/service/item_service.rs](mdc:src/application/service/item_service.rs)** - 実装例
- **[src/application/service/user_service.rs](mdc:src/application/service/user_service.rs)** - 実装例
- **[REMAINING_IMPROVEMENTS.md](mdc:REMAINING_IMPROVEMENTS.md)** - Phase 3完了状況

---

このガイドに従うことで、一貫性のあるエラーハンドリングとメトリクス記録を実現し、保守性と観測性の高いRust WebAPIを構築できます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kenken-ishiken) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
