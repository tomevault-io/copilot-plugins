## kubernetes-event-driven-service

> - **イベント駆動型マイクロサービスアーキテクチャ**を最小構成で検証

# 🚀 Gin + Kafka マイクロサービス PoC 設計書 (全サービス雛形 + K8s マニフェスト)

## 🎯 目的

- **イベント駆動型マイクロサービスアーキテクチャ**を最小構成で検証
- 各サービスは Go + Gin で実装
- Kafka (Strimzi) を利用してイベント連携
- ローカル (kind/minikube) → AWS EKS へ同じ構成でデプロイ可能

---

## 🛠️ 使用技術

### 共通

- 言語: **Go 1.21+**
- Web Framework: **Gin**
- Kafka Client: **github.com/segmentio/kafka-go**
- WebSocket: **github.com/gorilla/websocket** (Status Service)
- コンテナ化: **Docker**
- オーケストレーション: **Kubernetes (kind → EKS)**
- Kafka: **Strimzi Operator** (PLAINTEXT, 認証なし PoC 構成)

### フロントエンド (別途実装予定)

- Next.js + Tailwind CSS

---

## 📦 サービス一覧と役割

### 1. Order Service

- `POST /order`
- Kafka → `orders` トピックに `OrderCreated` を Publish

### 2. Inventory Service

- Consume: `OrderCreated`
- 在庫確認（インメモリ）
- Produce: `InventoryConfirmed` または `InventoryRejected`

### 3. Payment Service

- Consume: `InventoryConfirmed`
- ダミー決済処理
- Produce: `PaymentCompleted` または `PaymentFailed`

### 4. Notification Service

- Consume: `PaymentCompleted`
- Produce: `NotificationSent`

### 5. Shipping Service

- Consume: `PaymentCompleted`
- Produce: `Shipped`

### 6. Status Service

- Consume: 全イベント (`orders`, `inventory`, `payment`, `notification`, `shipping`)
- 注文 ID ごとの状態をインメモリ管理
- REST API: `GET /status/:orderId`
- WebSocket: `/ws/:orderId` (リアルタイム更新通知)

---

## 🔗 イベントフロー

```
OrderCreated ──> Inventory Service ──> InventoryConfirmed ──> Payment Service ──> PaymentCompleted ──> [ Notification Service, Shipping Service ]

```

---

## 📂 ディレクトリ構成 (モノレポ)

```
/repo
  /services
    /order-service
    /inventory-service
    /payment-service
    /notification-service
    /shipping-service
    /status-service
  /deploy
    /k8s
      /order-service
      /inventory-service
      /payment-service
      /notification-service
      /shipping-service
      /status-service
      /kafka

```

---

## 🚢 Kubernetes マニフェスト要件

### 共通

- 各サービス:
    - Deployment (replicas: 1)
    - Service (type: ClusterIP)
- Kafka:
    - Strimzi Operator を使用
    - 1ブローカー構成 (PLAINTEXT, 認証なし)
- Status Service:
    - `NodePort` または `Ingress` で外部アクセス可能に

### 環境

- **ローカル**: kind/minikube
- **AWS**: EKS (Strimzi Operator 再利用)

---

## 📝 制約・注意点

- DB は使用せず **インメモリ状態管理**
- Kafka 認証は **なし (PLAINTEXT)**
- CI/CD (GitHub Actions → ECR/EKS) は今回スコープ外
- フロントエンド (Next.js) は別途追加予定

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naonao0001777)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naonao0001777)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
