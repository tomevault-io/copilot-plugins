## electronbase

> このファイルは Claude Code・Amazon Q などの AI コーディングアシスタントが

# プロジェクト設計指針（AI アシスタント向け）

このファイルは Claude Code・Amazon Q などの AI コーディングアシスタントが
コードを読み書きする際に従うべき基本概念・ルールをまとめたものです。
実装前に必ず参照し、逸脱する場合はユーザーに確認を取ること。

---

## スタック概要

| レイヤー | 技術 | 役割 |
|---------|------|------|
| デスクトップ | Electron + React + TypeScript | フロントエンド |
| 認証 | Firebase Authentication | ログイン管理 |
| リモート DB（メイン） | TiDB（MySQL互換） | 永続データ全般 |
| リモート DB（補助） | Firebase Realtime Database | リアルタイム同期が必要な場合のみ |
| ローカル DB | IndexedDB | オフライン対応・表示高速化 |
| バックエンド | Firebase Cloud Functions (Node.js 20) | API・Stripe・メール等 |
| 課金 | Stripe | サブスクリプション・従量課金・都度課金 |

---

## 1. データ保存の原則

### TiDB がメインストレージ

- **すべての永続データは TiDB に書く**。Firebase Realtime DB はリアルタイム性が必要な場合の補助のみ。
- TiDB へのアクセスは必ず Firebase Cloud Functions 経由（フロントエンドから直接接続しない）。

### Firebase Realtime DB の使用は要確認

リアルタイム更新（ `onValue` / `onChildAdded` 等のリスナー）が必要になった場合は、
**実装前にユーザーへ確認を取ること**。

```
✅ Realtime DB を使ってよい例
  - メンテナンス状態のリアルタイム反映
  - お知らせの即時配信
  - ウォレット認証の nonce 管理（一時データ）

❌ Realtime DB を使わない例
  - ユーザー情報・課金情報 → TiDB
  - 使用量・ログ → TiDB
  - 設定値 → TiDB
```

### IndexedDB はローカルキャッシュ

- **表示に使うデータは IndexedDB から読む**（ページ表示の高速化・オフライン対応）。
- IndexedDB は TiDB / Realtime DB のミラーであり、単独の正として扱わない。

---

## 2. 書き込み順序（必ず守ること）

```
[ユーザー操作]
      ↓
[1] TiDB（または Realtime DB）に書き込む
      ↓ 成功を確認
[2] IndexedDB に書き込む
      ↓
[UI 更新]
```

- リモートへの書き込みが失敗した場合、IndexedDB には書かない。
- IndexedDB への書き込みが失敗してもリモートのデータは保持されているため、
  次回起動時の同期で回復する（エラーログだけ出して握りつぶさない）。
- 読み取り専用データ（マスターデータ等）は Realtime DB → IndexedDB の順で同期。

---

## 3. 実装方針：シンプル優先

> **パフォーマンスよりシンプルさを先に選ぶ。**

- 最適化は計測してボトルネックが判明してから行う。
- 抽象化・汎用化は「3回同じコードを書いたとき」に初めて検討する。
- 型・インターフェースは明示的に定義し、`any` は使わない。
- 非同期処理は `async/await` で統一（Promise チェーンは使わない）。
- エラーは握りつぶさず、必ずログに出すか上位に伝搬させる。

```typescript
// ❌ 避ける
try {
  await someOperation();
} catch {
  // 何もしない
}

// ✅ 推奨
try {
  await someOperation();
} catch (error) {
  logger.error('someOperation failed', error);
  throw error; // または UI にエラーを伝える
}
```

### フォールバックは必ずエラーにする

代替値・デフォルト値で「とりあえず動かす」フォールバックは禁止。
**フォールバックが必要な場面は、そもそも設計・データが壊れているサインである。**
エラーとして扱い、原因を特定・修正すること。

```typescript
// ❌ 禁止：エラーを隠してデフォルト値で続行
const planKey = billing?.plan_key ?? 'default_plan'; // なぜ plan_key がない？を調査すべき

// ❌ 禁止：空配列・null で握りつぶす
const users = await fetchUsers().catch(() => []); // エラーが消えて原因不明になる

// ✅ 推奨：エラーとして伝搬させる
const planKey = billing?.plan_key;
if (!planKey) throw new Error('billing.plan_key が取得できません。billing データを確認してください。');

// ✅ 推奨：エラーをログに残して再スロー
const users = await fetchUsers().catch((error) => {
  logger.error('fetchUsers failed', error);
  throw error;
});
```

**例外：UI 表示の初期値**（ローディング中の空文字・0 など）はフォールバックを許容する。
ただしその場合も、データ取得失敗時は必ずエラー状態を UI に伝えること。

---

## 4. 仮実装・デバッグコードの扱い

> **「とりあえず」の実装は禁止。**

### 原則

- 動作確認・エラー調査目的の仮コードは書いてよいが、**必ずユーザーに告知すること**。
- 調査が終わったら**必ず元に戻すか、正式に実装し直すこと**。
- 仮コードをコミットする場合は `TODO:` コメントを必ず付け、内容・期限・担当を明記する。

### 告知フォーマット

仮実装を行う際はユーザーへ以下を伝える:

```
⚠️ 仮実装: [何を仮にしたか]
理由: [なぜ仮にしたか（調査中・未実装等）]
戻すタイミング: [調査後 / XX実装後 / 次セッション等]
```

### コード上のマーク

```typescript
// TODO(仮): ここは hoge が未実装のため固定値。hoge 実装後に差し替えること
const planKey = 'standard_metered';

// TODO(デバッグ用): エラー調査のため console.log を追加。調査後に削除すること
console.log('[DEBUG] billing response:', response);
```

### やってはいけない仮実装の例

```typescript
// ❌ 黙って空配列を返す（エラーが消える）
return [];

// ❌ 黙って固定値を返す（バグの原因になる）
return { status: 'free' };

// ❌ コメントなしで catch を空にする（エラーが追えなくなる）
} catch (_e) {}

// ❌ 型を any にして後回し（型エラーが消えて問題が見えなくなる）
const data = response as any;
```

---

## 5. ログ出力

### 環境別ログレベル

| 環境 | 出力するログ |
|------|------------|
| 開発（development） | すべて（debug / info / warn / error） |
| 本番（production） | **error のみ** |

### logger の使い方

`LoggingService` を使う。`console.log` を直接書かない。

```typescript
import { logger } from '../services/LoggingService';

logger.debug('詳細情報');   // 開発のみ出力
logger.info('処理完了');    // 開発のみ出力
logger.warn('注意');        // 開発のみ出力
logger.error('エラー', e);  // 本番でも出力
```

`LoggingService` は `VITE_ENV` 環境変数を参照してレベルを切り替える:
- `VITE_ENV=development` → 全レベル出力
- `VITE_ENV=production` → error のみ

### Cloud Functions 側

```typescript
// functions/src/ 内では console の代わりに functions.logger を使う
import * as functions from 'firebase-functions';

functions.logger.info('処理開始');   // 開発のみ出力したい場合は env 判定を追加
functions.logger.error('エラー', e); // 常に出力
```

---

## 6. ディレクトリ構成（重要ファイル）

```
ElectronBase/
├── CLAUDE.md                    # ← このファイル（AI アシスタント向け指針）
├── SETUP.md                     # 初回セットアップ手順
├── SETUP.RUNNING.md             # デプロイ・ビルド・運用手順
├── db/
│   ├── realtime-db-rules.json   # Realtime DB ルール（編集元）
│   ├── realtime-db-schema.md    # Realtime DB スキーマ定義
│   └── tidb-create.sql          # TiDB CREATE文
├── frontend/                    # Electron アプリ（React）
│   └── src/
│       ├── main/                # Electron メインプロセス
│       ├── preload/             # コンテキストブリッジ
│       └── renderer/
│           ├── services/        # ビジネスロジック・API呼び出し
│           ├── components/      # UI コンポーネント
│           ├── types/           # 型定義
│           └── i18n/            # 多言語対応
├── functions/                   # Firebase Cloud Functions
│   └── src/
│       ├── index.ts             # エントリポイント・認証系
│       ├── tidb.ts              # TiDB 接続・クエリ
│       └── *Api.ts              # 機能別 API（billingApi.ts 等）
├── admin/                       # 管理ダッシュボード（React）
└── firebase.json                # Firebase 設定
```

---

## 7. よくある実装パターン

### Cloud Functions の API 追加

`functions/src/versionApi.ts` を参考にする。パターン:
1. `handle*` 関数（内部実装）
2. 末尾に `export const functionNameDev/Prd` を追加
3. `index.ts` で re-export
4. `firebase.json` の hosting rewrite に追加

### フロントエンドからの API 呼び出し

```typescript
// FirebaseService.ts の API_TO_FUNCTION マップに追加
// ↓
const url = getApiUrl('functionName');
const idToken = await getFirebaseAuth().currentUser?.getIdToken();
const res = await fetch(url, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${idToken}` },
  body: JSON.stringify({ ... }),
});
```

### IndexedDB への書き込み

```typescript
// リモート書き込み成功後に実行
await tidbApi.updateSomething(data);      // 1. リモート
await IndexedDBService.saveSomething(data); // 2. ローカル
```

---

## 8. 実装前のチェックリスト

新機能を実装する前に以下を確認すること:

- [ ] データの永続先は TiDB か？（Realtime DB を使う場合はユーザーに確認）
- [ ] 書き込み順序は「リモート → IndexedDB」になっているか？
- [ ] `console.log` ではなく `logger` を使っているか？
- [ ] エラーを握りつぶしていないか？
- [ ] WEB標準の機能か。ELCTORN専用の機能使っていないか？
- [ ] フォールバックで `null` / 空配列 / デフォルト値を黙って返していないか？
- [ ] 仮実装・デバッグコードにはユーザーへの告知と `TODO(仮):` コメントがあるか？
- [ ] 型定義はあるか？（`any` を使っていないか？）
- [ ] 過度な抽象化・早すぎる最適化をしていないか？

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matome2009) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
