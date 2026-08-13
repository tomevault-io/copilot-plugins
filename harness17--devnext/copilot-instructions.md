## devnext

> このファイルはClaude Code・Codex・Cursor等のAIエージェントが参照する共通ルールです。

# DevNext — Agent Guidelines

このファイルはClaude Code・Codex・Cursor等のAIエージェントが参照する共通ルールです。
Claude Code固有の設定（スキル・フック）は `.claude/` を参照してください。

---

## プロジェクト概要

- **種別**: ASP.NET Core 10 MVC Webアプリ（ポートフォリオ兼テンプレート）
- **目的**: 新案件の出発点となるコアテンプレート（認証・ユーザー管理・基本CRUD）
- **言語**: C# / Razor Views / JavaScript (jQuery)

## プロジェクト構成

```
DevNext/          ← メインWebアプリ（RootNamespace: Site）
CommonLibrary/    ← 共通ライブラリ（RootNamespace: Dev.CommonLibrary）
BatchSample/      ← バッチ処理サンプル（RootNamespace: BatchSample）
Tests/            ← xUnit テストプロジェクト
Samples/          ← 独立したサンプルプロジェクト群（各自独立）
docs/             ← 設計書・実装計画
scripts/          ← 開発補助スクリプト
```

### Samples/ の一覧

| フォルダ | 概要 | ポート |
|---------|------|--------|
| `ApiSample` | REST API + JWT 認証 + Swagger | 5042 |
| `ApiClientSample` | ApiSample を呼び出す MVC クライアントサイト | 5170 |
| `DatabaseSample` | EF Core CRUD サンプル | — |
| `ExcelSample` | Excel インポート・エクスポート | — |
| `FileSample` | ファイルアップロード・ダウンロード | — |
| `MailSample` | メール送信（smtp4dev） | — |
| `PdfSample` | PDF 生成 | — |
| `WizardSample` | 複数ステップフォーム | — |

### Samples/ の設計制約

- 各Sampleは `CommonLibrary` を参照してよい
- **Sample同士は依存しない**（FileSampleがMailSampleを参照するなど禁止）
- 各Sampleは単独でビルド・起動できる状態にすること
- Sample内のエンティティ・DBContextはSample専用とし、コアのDBContextとは分離する

---

## コーディング方針

- **メンテナンス性を最優先**とする
- **可能な限りコメントを生成**する

### 名前空間ルール

| プロジェクト | RootNamespace |
|---|---|
| `DevNext/` | `Site` |
| `CommonLibrary/` | `Dev.CommonLibrary` |
| `BatchSample/` | `BatchSample` |

### エンティティ設計ルール

- すべてのエンティティは **`SiteEntityBase` を継承**すること（`Id: long` + 共通監査カラムを統一するため）
- 更新履歴が必要なエンティティは以下のパターンで実装すること：

```
XxxEntityBase (abstract) : SiteEntityBase
  ├── XxxEntity          // 本体テーブル
  └── XxxEntityHistory   // 履歴テーブル（HistoryId: long [Key], IEntityHistory）
```

### DI 登録ルール

- `[ServiceFilter(typeof(XxxAttribute))]` を使う場合、対象クラスを `Program.cs` で `AddScoped` 登録する

```csharp
builder.Services.AddScoped<AccessLogAttribute>();
```

### パスワードポリシー

`Program.cs` で設定。以下をすべて満たすこと：最低6文字・大文字・小文字・数字・記号すべて必須

### テスト記述時の注意

`SignInResult` は `Microsoft.AspNetCore.Identity.SignInResult` と `Microsoft.AspNetCore.Mvc.SignInResult` が競合するため、テストコードではエイリアスを使用すること：

```csharp
using IdentitySignInResult = Microsoft.AspNetCore.Identity.SignInResult;
```

---

## データベースルール

- **SQL Server**、DB名: `DevNextDB`、接続文字列キー: `SiteConnection`
- 接続設定: 各プロジェクトの `appsettings.json`（Windows 認証 + `TrustServerCertificate=True` を使うため機密情報なし。本番環境は環境変数 `ConnectionStrings__SiteConnection` で上書きする）
- **EF Core Migrations** を使用してスキーマ管理する
- DB 作成・マイグレーション適用・Seed 投入はアプリ起動時に自動実行される（`Program.cs` の `MigrateAsync` + `SeedAsync`）
- テーブル・カラムを追加・変更した場合は `/add-entity` スキルを使用すること

### Seed データ（初期ユーザー）

| UserName | Email | Password | Role |
|---|---|---|---|
| admin1@sample.jp | admin1@sample.jp | Admin1! | Admin |
| member1@sample.jp | member1@sample.jp | Member1! | Member |

---

## コマンドリファレンス

| 用途 | コマンド |
|------|---------|
| ビルド | `dotnet build DevNext.slnx` |
| 開発サーバー起動 | `cd DevNext && dotnet run` |
| テスト実行 | `cd Tests && dotnet test` |
| マイグレーション追加 | `dotnet ef migrations add <名前> --project DevNext` |
| DB に適用（手動） | `dotnet ef database update --project DevNext` |

---

## 実装計画の読み方

実装タスクは `docs/plans/YYYY-MM-DD-<feature>.md` に保存されています。
作業開始時は必ず該当する計画ファイルを読み、チェックボックス（`- [ ]`）を順番に実行してください。

補足:
- 実装計画の正式な配置先は `docs/plans/`
- 古い資料に旧パスの記載が残っていても、実ファイル配置は `docs/plans/...` を優先する

---

## 共同開発ハーネス（Codex × Claude Code）

Codex は作業開始時に `CLAUDE_CODE_HANDOFF.md` の最新セクションを読み、`.agents/skills/implement-task/SKILL.md` と `.claude/rules/project-collaboration-profile.md` に従って作業する。

Claude Code から Codex へ振る場合は `CLAUDE_CODE_HANDOFF.md` に目的、完成条件、触ってよい範囲、verify コマンド、レビュー観点を追記する。反対側レビューは `.claude/skills/cross-review/SKILL.md` と `.claude/rules/handoff-protocol.md` に従う。Codex 側からスクリプトで自動トリガーする場合は `scripts/invoke-claude-review.ps1` を使用する。

merge 前はセルフ verify、相互レビュー、重大指摘なし、ユーザーの明示指示の 4 条件を揃える。

---

## アドバイザー戦略（モデル選択）

Sonnet（Executor）がタスク全体を実行し、判断が難しい局面でのみ Opus（Advisor）に相談する。

| 役割 | モデル | 担当 |
|------|--------|------|
| Executor | Sonnet | 実装・ファイル操作・ビルド・テスト |
| Advisor | Opus | 方針・トレードオフの判断のみ（実行しない） |

**Opus に相談するタイミング**:
- アーキテクチャ判断（CommonLibrary か呼び出し側か、など）
- 複数の実装方針のトレードオフ比較
- セキュリティ設計の妥当性確認
- 根本原因が特定できない複雑なバグの仮説出し

**相談しないタイミング**: ルーティン実装・ビルドエラー修正・ファイル操作・Git 操作

詳細: `.claude/rules/advisor-strategy.md`

---

## ⛔ 禁止操作

以下の操作は絶対に実行しないこと。

| 禁止コマンド | 理由 |
|-------------|------|
| `git reset --hard` | コミット済みの作業が失われる |
| `git push --force` | リモートの履歴を破壊する |
| `git clean -f` / `git clean -fd` | 未追跡ファイルを削除する |
| `git add -A` / `git add .` | 意図しないファイルが混入する可能性がある |
| `rm -rf <ディレクトリ>` | 復元不可能な削除になる |

追加の制約:
- テストが失敗したままコミットしない
- ビルドエラーが残ったままコミットしない
- `appsettings.Development.json` はコミットしない（`.gitignore` 対象）

---

## ⚠️ 実装完了後の必須手順

実装が完了したら必ず以下を順番に実行してください：

1. ビルドが通ることを確認: `dotnet build DevNext.slnx`
2. テストが通ることを確認: `cd Tests && dotnet test`
3. `git add` で変更ファイルを個別指定してコミット（`git add -A` は使用しない）

補足:
- `scripts/request-review.ps1` は今後の標準フローでは使用しない
- レビューが必要な場合は、別途その時点の運用ルールに従うこと

---
> Source: [harness17/DevNext](https://github.com/harness17/DevNext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
