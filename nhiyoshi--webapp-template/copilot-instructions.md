## webapp-template

> FastAPI実装のためのベストプラクティスと標準


# FastAPIベストプラクティス

@global.mdc

## API設計とルーティング

1. **ルーティング構造**
   - 論理的なリソースごとにルーターを分割する
   - 明確なURL構造と命名規則を使用する (`/users/{user_id}/items/{item_id}`)
   - CRUDオペレーションには一貫したHTTPメソッドを使用する (GET, POST, PUT, DELETE)
   - エンドポイントのタグ付けを活用してSwaggerドキュメントを整理する

2. **パスオペレーション関数**
   - 関数名は明確な動詞+名詞の形式にする (`get_user`, `create_item`)
   - 関数のドキュメント文字列で機能を説明する
   - パスパラメータ、クエリパラメータ、リクエストボディを適切に区別する
   - 適切なHTTPステータスコードを常に返す

```python
@router.get("/{item_id}", response_model=ItemResponse, status_code=status.HTTP_200_OK)
async def get_item(
    item_id: int,
    current_user: User = Depends(get_current_user)
) -> ItemResponse:
    """
    特定のアイテムを取得する。

    - item_id: 取得するアイテムのID
    - current_user: 認証されたユーザー

    存在しない場合はHTTP 404を返す
    """
    item = await items_service.get_by_id(item_id)
    if not item:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Item not found")
    return item
```

## Pydanticモデル

1. **モデル設計**
   - リクエストモデルとレスポンスモデルを分離する
   - 継承を利用して重複を減らす (`ItemBase`, `ItemCreate`, `ItemResponse`)
   - フィールドにはバリデーションと説明を含める
   - 機密データは除外するようにモデルを設計する

2. **バリデーション**
   - Pydanticのバリデーションを最大限に活用する
   - カスタムバリデータを作成してビジネスルールを強制する
   - エラーメッセージをユーザーフレンドリーにする

```python
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, description="ユーザー名")
    email: EmailStr = Field(..., description="メールアドレス")
    password: str = Field(..., min_length=8, description="パスワード")

    @validator('username')
    def username_alphanumeric(cls, v):
        if not v.isalnum():
            raise ValueError('ユーザー名にはアルファベットと数字のみ使用できます')
        return v
```

## 依存性注入

1. **依存関係の設計**
   - 共通の機能には依存性関数を作成する
   - スコープを適切に設定する (request, session, app)
   - パラメータ依存関係と関数依存関係を適切に使い分ける
   - コード重複を避けるために依存関係を利用する

2. **認証と認可**
   - JWT認証を実装する場合は依存関係を使用する
   - ロールベースのアクセス制御を依存関係に組み込む
   - エラーハンドリングを明確に定義する

```python
def get_current_active_user(
    current_user: User = Depends(get_current_user),
) -> User:
    if not current_user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="アカウントが非アクティブです",
        )
    return current_user
```

## 非同期処理

1. **非同期原則**
   - I/O束縛の操作には`async/await`を使用する
   - データベースやその他の外部サービスとの通信は非同期にする
   - CPUバウンドの処理には別のスレッドプールを利用する
   - 適切なエラーハンドリングを非同期関数に実装する

2. **パフォーマンス最適化**
   - 複数の非同期操作を並行して実行するために`asyncio.gather`を使用する
   - 長時間実行される処理にはバックグラウンドタスクを検討する
   - リクエストコンテキスト内で状態を適切に管理する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nhiyoshi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
