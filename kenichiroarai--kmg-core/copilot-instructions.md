## kmg-core

> - JUnit Jupiter: 5.10.2

# 技術スタック

## コア技術

- Java: 21
- Apache POI: 5.2.5
- JUnit Jupiter: 5.10.2
- Mockito: 5.11.0

## 開発ツール

- Maven: 3.12.1（コンパイラプラグイン）
- JaCoCo: 0.8.11
- Maven Surefire Plugin: 3.2.5

## プロジェクト設定

- Java Compiler: Java 21
- Source Encoding: UTF-8
- JavaFX: 21.0.2

---

## プロジェクト構成

以下のディレクトリ構造に従って実装を行ってください：

```bash
kmg-core/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── kmg/
│   │   │       └── core/
│   │   │           ├── domain/           # ドメイン層
│   │   │           │   ├── service/     # ドメインサービス
│   │   │           │   │   └── impl/   # サービス実装
│   │   │           │   └── types/      # ドメイン型定義
│   │   │           └── infrastructure/   # インフラストラクチャ層
│   │   │               ├── common/      # 共通機能
│   │   │               ├── exception/   # 例外クラス
│   │   │               ├── model/       # モデル
│   │   │               │   └── impl/   # モデル実装
│   │   │               ├── type/        # 型定義
│   │   │               ├── types/       # 型定義（列挙型など）
│   │   │               │   └── template/ # テンプレート型
│   │   │               └── utils/       # ユーティリティ
│   │   └── resources/                    # リソースファイル
│   └── test/
│       ├── java/                         # テストコード
│       │   └── kmg/
│       │       └── core/
│       │           ├── domain/           # ドメインテスト
│       │           │   ├── service/     # サービステスト
│       │           │   │   └── impl/   # 実装テスト
│       │           │   └── types/      # 型定義テスト
│       │           ├── infrastructure/   # インフラストラクチャテスト
│       │           │   ├── exception/   # 例外テスト
│       │           │   ├── model/       # モデルテスト
│       │           │   │   └── impl/   # 実装テスト
│       │           │   ├── type/        # 型テスト
│       │           │   ├── types/       # 型定義テスト
│       │           │   │   └── template/ # テンプレートテスト
│       │           │   └── utils/       # ユーティリティテスト
│       │           └── test/            # テストユーティリティ
│       └── resources/                    # テストリソース
│           └── kmg/
│               └── core/
│                   └── domain/
│                       └── model/
│                           └── impl/
│                               └── kmg_sql_path_model_impl_tests/
├── scripts/                              # スクリプトファイル
├── doc/                                  # ドキュメント
├── .settings/                            # Eclipse設定
├── .vscode/                              # VSCode設定
├── .github/                              # GitHub設定
│   └── workflows/                        # GitHub Actions
├── target/                               # ビルド成果物
├── pom.xml                               # Mavenプロジェクト設定
├── .gitignore                            # Git除外設定
├── .project                              # Eclipseプロジェクト設定
├── .classpath                            # Eclipseクラスパス設定
├── .tgitconfig                           # TortoiseGit設定
├── LICENSE                               # ライセンスファイル
└── README.md                             # プロジェクト説明
```

### 配置ルール

- ドメインクラス → `src/main/java/kmg/core/domain/`
- インフラストラクチャクラス → `src/main/java/kmg/core/infrastructure/`
- プレゼンテーションクラス → `src/main/java/kmg/core/presentation/`
- テストクラス → 対応する実装クラスと同じパッケージ構成

### テスト要件

- JUnitテストレポートの生成
- カバレッジレポートの生成（JaCoCo）
- メソッドの複雑度は15以下に制限

---

## 順守

以上の内容を順守し、タスクを遂行してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KenichiroArai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
