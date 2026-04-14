## aithlete

> - コーチの回答や過去のチャット履歴、関連ドキュメントをRAG（Retrieval-Augmented Generation）で検索・抽出し、その情報をもとにAI（大規模言語モデル）が回答を生成します。

--- Context from: GEMINI.md ---
## AIコーチの回答生成について


- コーチの回答や過去のチャット履歴、関連ドキュメントをRAG（Retrieval-Augmented Generation）で検索・抽出し、その情報をもとにAI（大規模言語モデル）が回答を生成します。
- 具体的には、ユーザーからの質問やチャット文脈をもとに、まず関連するコーチの過去回答やナレッジベースを検索（retrieval）し、その内容をプロンプトに含めてAIが自然な回答文を生成（generation）します。
- これにより、AIコーチは実際のコーチの知見や過去のやりとりを活用した、より信頼性・一貫性の高い回答を返すことができます。
- RAGの導入により、AIの回答品質向上・パーソナライズ・ドメイン知識の反映が可能となります。


### ポーズ推定（Pose Estimation）とフォーム指導
- 人間の動画や画像から骨格（スケルトン）情報を抽出する「ポーズ推定（Pose Estimation）」技術を活用し、フォーム指導を行います。
- 具体的には、OpenPoseやMediaPipe Poseなどのライブラリを用いて、画像・動画からキーポイント（関節座標）やスケルトンデータを取得します。
- モデルの独自学習には多くのデータと高精度なアノテーションが必要なため、まずは抽出したポーズ情報（ベクトルデータ）と既存のフォーム指導ナレッジをRAG（Retrieval-Augmented Generation）的に組み合わせてAIによるアドバイス生成を行います。
- これにより、ユーザーの動作と理想的なフォームの類似度や特徴点の違いをもとに、パーソナライズされたフィードバックを提供します。

---

## ドメインモデル（クラス図）

以下は`class.md`で定義されたドメインモデルのMermaidクラス図です。

```mermaid
classDiagram
	class Appointment {
		+ID: string
		+ChatID: string
		+Title: string
		+Description: string
		+ScheduledAt: time.Time
		+Duration: int
		+Status: string
		+CreatedAt: time.Time
		+UpdatedAt: time.Time
	}

	class AppointmentParticipant {
        +AppointmentID: string
        +ParticipantID: string
        +Status: string
    }

	class Chat {
		+ID: string
		+StartedAt: time.Time
		+LastActiveAt: time.Time
		+Title: string?
		+Questions: Question[]
		+Answers: Answer[]
		+ParticipantIDs: string[]
	}

	class Notification {
		+ID: string
		+UserID: string
		+ChatID: string
		+Type: string
		+Content: string
		+CreatedAt: string %% datetime型推奨
		+Read: bool
	}

	class Participant {
		+ID: string
		+Name: string
		+Email: string
		+Role: string %% "coach", "user", "ai_coach" など
		+Sports: string[]
		+IconURL: string? %% *string（オプショナル）
	}

	class Session {
	+ID: string
	+ParticipantID: string
	+ChatID: string
	+StartedAt: string
	+LastActiveAt: string
	+Status: string %% "active", "inactive", "disconnected" など
	+IPAddress: string
	}

	class Question {
		+ID: string
		+ChatID: string
		+ParticipantID: string
		+Content: string
		+Attachments: Attachment[]
		+CreatedAt: time.Time
	}
	class Answer {
		+ID: string
		+ChatID: string
		+QuestionID: string
		+ParticipantID: string
		+Content: string
		+Attachments: Attachment[]
		+CreatedAt: time.Time
	}
	class Attachment {
		+ID: string
		+Type: string
		+URL: string
		+Thumbnail: string? 
		+PoseID: string? 
		+Pose: PoseData? 
		+Meta: string 
		+OriginalID: string? 
		+Original: Attachment?
		+QuestionID: string? 
		+AnswerID: string? 
	}

	class PoseData {
		+ID: string
		+ParticipantIDs: string[]
		+Score: float64
	}
	Chat "1" -- "0..*" Session

	Participant "1" -- "0..*" Session

	Participant "1" -- "*" Notification
	Chat "1" <.. "*" Notification

	Chat "*" -- "*" Participant
	Chat "1" o-- "*" Question 
	Chat "1" o-- "*" Answer
	Question "1" -- "0..*" Attachment
	Answer "1" -- "0..*" Attachment
	Attachment "1" -- "0..1" PoseData

	Chat "1" -- "0..*" Appointment
    Appointment "1" -- "1..*" AppointmentParticipant
    Participant "1" -- "0..*" AppointmentParticipant
```

---
本リポジトリは、ドメイン駆動設計（DDD）およびオニオンアーキテクチャをベースに、マイクロサービスごとにコンテナ分割できる構成を推奨します。

---

## コーディング規約

- **コメントの原則**: ソースコード中に、古いコードの残骸などをコメントアウトとして残すことは原則として避けます。コードは常にクリーンな状態を保ちます。
- **自己説明的なコード**: コード自体がドキュメントとして機能するような、明確で読みやすい命名や構造を心がけます。これにより、コードの意図を理解するためにコメントに頼る必要性を減らします。

---

## システム構成図（サービス間連携）

```mermaid
flowchart LR
	frontend["frontend"]
	api["api"]
	db["db:Postgres"]
	broker["broker:Redis"]
	ws["ws"]
	rag["rag"]
	embedding["embedding"]
	weaviate["weaviate"]

	frontend -- REST/API --> api
	api -- PubSub --> broker
	api -- DB --> db
	ws -- PubSub --> broker
	api -- RAG --> rag
	rag -- EmbeddingAPI --> embedding
	rag -- VectorSearch --> weaviate
	api -- VectorStore --> weaviate
```


## リアルタイム通信設計まとめ

### チャットのリアルタイム通信設計

- チャット画面がアクティブな場合、クライアントは常にWebSocketサーバーに接続し、送信・受信ともにWSイベントでリアルタイム通信を行う。
- 送信側はWS経由でメッセージ送信。受信側がアクティブなら即時UI反映、非アクティブならDB保存＋OSプッシュ通知（APNs/FCM等）で通知。
- 受信者が次回アクティブになった時、DBから未読メッセージをfetchして同期する。
- REST APIのAdd系（例: AddQuestion）は主な操作では使われず、WS未接続時のフォールバックや管理用途など補助的な役割。

### pushとfetchの併用

- push（WebSocket等）：サーバー→クライアントへの能動的なリアルタイム通知。即時UI反映。
- fetch（REST等）：クライアント→サーバーへの明示的なデータ取得。整合性担保や再取得に利用。
- pushだけでは取りこぼしやズレのリスクがあるため、必要に応じてfetchで正規データを取得する。

---

## 推奨ディレクトリ構成

```
/api              # REST APIサービス（Goなど）
  /domain         # ドメイン層（エンティティ・値オブジェクト・集約・リポジトリIF・ドメインサービス）
  /application    # アプリケーション層（ユースケース・アプリケーションサービス・DTO）
  /infrastructure # インフラ層（DB実装・外部API・リポジトリ実装）
  /presentation   # プレゼンテーション層（HTTPハンドラ・ルーティング・認証）
  main.go         # エントリポイント
/frontend         # フロントエンド（Next.js, React等）
/db               # DBマイグレーション・DDL等
/ai               # AI/ML関連サービス
/pose-estimator   # 姿勢推定サービス

uml_component_diagram.md # アーキテクチャ図・UML
README.md                # このファイル
```

- サービスごとにディレクトリを分割し、各サービスは独立したコンテナとしてデプロイ可能です。
- 各サービス内でオニオンアーキテクチャのレイヤー分離を徹底します。
- 共通仕様やAPI設計はこのREADMEやuml_component_diagram.mdで統一管理してください。
--- End of Context from: GEMINI.md ---

## システム構成図（サービス間連携）

```mermaid
flowchart LR
	frontend["frontend"]
	api["api"]
	db["db:Postgres"]
	broker["broker:Redis"]
	ws["ws"]
	rag["rag"]
	embedding["embedding"]
	weaviate["weaviate"]

	frontend -- REST/API --> api
	api -- PubSub --> broker
	api -- DB --> db
	ws -- PubSub --> broker
	api -- RAG --> rag
	rag -- EmbeddingAPI --> embedding
	rag -- VectorSearch --> weaviate
	api -- VectorStore --> weaviate
```


## リアルタイム通信設計まとめ

### チャットのリアルタイム通信設計

- チャット画面がアクティブな場合、クライアントは常にWebSocketサーバーに接続し、送信・受信ともにWSイベントでリアルタイム通信を行う。
- 送信側はWS経由でメッセージ送信。受信側がアクティブなら即時UI反映、非アクティブならDB保存＋OSプッシュ通知（APNs/FCM等）で通知。
- 受信者が次回アクティブになった時、DBから未読メッセージをfetchして同期する。
- REST APIのAdd系（例: AddQuestion）は主な操作では使われず、WS未接続時のフォールバックや管理用途など補助的な役割。

### pushとfetchの併用

- push（WebSocket等）：サーバー→クライアントへの能動的なリアルタイム通知。即時UI反映。
- fetch（REST等）：クライアント→サーバーへの明示的なデータ取得。整合性担保や再取得に利用。
- pushだけでは取りこぼしやズレのリスクがあるため、必要に応じてfetchで正規データを取得する。

---

## 推奨ディレクトリ構成

```
/api              # REST APIサービス（Goなど）
  /domain         # ドメイン層（エンティティ・値オブジェクト・集約・リポジトリIF・ドメインサービス）
  /application    # アプリケーション層（ユースケース・アプリケーションサービス・DTO）
  /infrastructure # インフラ層（DB実装・外部API・リポジトリ実装）
  /presentation   # プレゼンテーション層（HTTPハンドラ・ルーティング・認証）
  main.go         # エントリポイント
/frontend         # フロントエンド（Next.js, React等）
/db               # DBマイグレーション・DDL等
/ai               # AI/ML関連サービス
/pose-estimator   # 姿勢推定サービス

uml_component_diagram.md # アーキテクチャ図・UML
README.md                # このファイル
```

- サービスごとにディレクトリを分割し、各サービスは独立したコンテナとしてデプロイ可能です。
- 各サービス内でオニオンアーキテクチャのレイヤー分離を徹底します。
- 共通仕様やAPI設計はこのREADMEやuml_component_diagram.mdで統一管理してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maehiyu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maehiyu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
