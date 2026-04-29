## densha-wasmcloud

> > 目的：UIを「純粋関数 × 状態DAG × 能力(capabilities)」として扱い、\*\*内容アドレス化（Merkle DAG）\*\*を軸に配信・検証・部分差し替えを可能にする。

# densha — Merkle DAG + WASM + hataori ベースのフロントエンド設計

> 目的：UIを「純粋関数 × 状態DAG × 能力(capabilities)」として扱い、\*\*内容アドレス化（Merkle DAG）\*\*を軸に配信・検証・部分差し替えを可能にする。

---

## 0. 要点まとめ

* **土台**：hataori（SWC プラグイン＋CAS）で TS/JS 関数をハッシュ化／依存解析 → **コードDAG** を生成。
* **WASM**：Rust/AssemblyScript/TinyGo など任意の WASM コンポーネントを同じ DAG に統合。
* **状態**：Snapshot（推奨）または OpLog（CRDT寄り）で **状態DAG** を管理。
* **副作用**：最小権限の **Ability** として注入（Clock/Random/Http/Storage/Crypto/Dom…）。
* **配信**：CAS/CDN に `/cas/<hash>` で永続キャッシュ。**ハッシュ = 真理**。

https://www.npmjs.com/package/hataori

---

## 1. フォルダ構造（Next.js の app 方式を踏襲）

```
app/
  layout.tsx             # ルートレイアウト（hataori 対象）
  page.tsx               # "/" ページ
  loading.tsx            # ストリーム用プレースホルダ
  error.tsx              # エラー UI

  about/
    page.tsx

  blog/
    layout.tsx
    page.tsx             # "/blog"
    [slug]/
      page.tsx
      loading.tsx
      generate.ts        # SSG/ISR 用の生成フック

  api/
    hello.action.ts        # Server Action（副作用境界）

shared/
  components/
    Button.tsx
  state/
    counter.state.ts
  abilities/
    http.ability.ts

densha.config.ts           # ルーティング/SSR/プリフェッチ等のポリシー
```

* `*.tsx`：hataori が AST 正規化→ハッシュ化。
* 動的セグメント：`[param]` / `[[...rest]]` をサポート。
* `generate.ts`：SSG/ISR 相当（後述）。
* `*.action.ts`：Server Ability を経由してのみ I/O を許可。

---

## 2. ルーティング規則

* マッチング：`/a/b` → `app/a/b/page.tsx`（なければ上位 `layout.tsx` と合成）。
* 優先順位：**静的 > 動的 > キャッチオール**。
* リンク：`<Link href="/p">` で **DAG 幅優先プリフェッチ**。
* ページ関数シグネチャ：`defComponent((params, generatedProps) => VNode)`。

**例：**

```ts
// app/blog/[slug]/page.tsx
import { defComponent, h } from "densha";
export default defComponent(({ slug }: { slug: string }, gp: { mdxHtml: string }) =>
  h("article", { dangerouslySetInnerHTML: gp.mdxHtml })
);
```

```ts
// app/blog/[slug]/generate.ts  — SSG/ISR
import { serverAbility } from "@/shared/abilities/http.ability";
export const prerender = {
  revalidateSeconds: 3600,
  paths: async () => serverAbility.listSlugs(),
  props: async ({ slug }) => {
    const md = await serverAbility.fetchPost(slug);
    return { mdxHtml: renderMDX(md) };
  }
};
```

* 生成結果は内容アドレス化された `props@<hash>.json` として保存 → CDN 永続キャッシュ。

---

## 3. データモデル（3つの DAG）

### 3.1 コードDAG（JS/WASM）

* **ノード**：`CodeNode { hash, kind: 'js'|'wasm', deps: Hash[], url }`
* **正規化**：JS は AST 正規化、WASM はバイナリ正規化（セクション順序/名前除去）→ blake3 等でハッシュ。

### 3.2 状態DAG

* **Snapshot（推奨）**：`StateSnap { hash, parents: Hash[], patch }`。
* **OpLog（拡張）**：`StateOp { id, parent[], payload, timestamp, auth }`（CRDT 寄り）。

### 3.3 Ability DAG（ポリシー）

* どの `u#` がどの Ability を使えるかを静的ポリシーとして別 DAG で管理。

---

## 4. ビルド & 配信パイプライン

1. **JS/TS**：SWC（hataori）で関数抽出→依存解析→AST 正規化→**関数ハッシュ(u#)** → CAS 格納 → `/cas/<hash>.mjs` として出力。
2. **WASM**：`wasm-pack`/`asc`/`tinygo` で最適化→**バイナリ正規化**→ `/cas/<hash>.wasm`。
3. **マニフェスト**（統合 DAG）

```json
{
  "root": "u#appRootHash",
  "nodes": {
    "u#appRootHash": { "kind": "js",   "deps": ["u#Layout","u#Routes"], "url": "/cas/u_appRootHash.mjs" },
    "u#CounterWasm": { "kind": "wasm", "deps": ["u#DomShim"],            "url": "/cas/u_counter.wasm", "abi": "densha-v1" }
  }
}
```

* 署名付きマニフェスト（任意）：サプライチェーン検証に利用。

---

## 5. ランタイム設計

### 5.1 ローダ

* `resolve(hash)`：マニフェスト→`fetch(url)`→**ハッシュ検証**→実体化。循環は DAG により禁止。

### 5.2 WASM ブリッジ（安定 ABI）

* WASM 側：`render(props_json) -> vdom_json` の純粋コア。
* JS 側：`props`/`vdom` を JSON/FlatBuffer で受け渡し。
* イベントはキュー化 → Ability 経由で副作用を実行。

### 5.3 レンダラ/スケジューラ

* MVP は VDOM（または軽量 Signals）。
* **ハッシュ・ホットスワップ**：`swapComponent(u#old, u#new)` で依存差し替え＆再描画。

### 5.4 状態 API（Snapshot 版）

```ts
import { useStateDAG } from "densha/state";
export const Counter = defComponent(() => {
  const [n, setN] = useStateDAG("counter", 0);
  return h("button", { onclick: () => setN(n + 1) }, `count=${n}`);
});
```

---

## 6. SSR は入れるべきか？

### 結論

* **入れるべき。ただし “決定性 SSR” を基本に、SSG/ISR とハイブリッド**。
* パーソナライズや重WASMは **CSR** へ逃がす。

### 理由

* 無条件 SSR は **ハッシュ不一致**／**WASM コールドスタート**／**キャッシュ劣化** の危険。
* それでも SSR は SEO/初期可読性/ストリーミングで有利。純粋・決定的に描ける範囲のみ SSR。

### 実行モード

* **Deterministic SSR（推奨）**：seed 固定、許可 Ability を限定、出力は `HTML + manifest + props@hash`。
* **Edge Streaming SSR（限定）**：DAG 幅優先で部分 HTML をプッシュ。非決定要素は CSR で整合。

---

## 7. 公開 API（DX）

```ts
// Component 定義
import { ability, defComponent, h } from "densha";
export const Button = defComponent(({ label }: { label: string }) => {
  const log = ability("Console");
  return h("button", { onclick: () => log("clicked") }, label);
});

// 起動
import { bootstrap } from "densha/runtime";
bootstrap({ root: "u#appRootHash", abilities: { Console: console.log }, manifestUrl: "/cas/manifest.json" });
```

---

## 8. 設定ファイル例

```ts
// densha.config.ts
export default {
  routing: { dir: "app", dynamic: ["[", "]"] },
  ssr: {
    mode: "deterministic",              // "edge-streaming" / "off"
    allowAbilities: ["Console"],
    seed: "2025-09-13T00:00:00Z#fixed"
  },
  prefetch: { strategy: "dag-width-first", budgetKB: 200 }
} as const;
```

---

## 9. ロードマップ

* **M0（POC）**：JS のみ／Snapshot 状態／Console & Clock の2能力／ハッシュ・ホットスワップ。
* **M1**：WASM ブリッジ v1（JSON ABI）、SSR 対応、DAG Explorer/Hot-Swap DevTools。
* **M2**：署名付きマニフェスト、Ability ポリシーDAG、プリフェッチ最適化、CRDT 実験（OpLog→Snapshot 圧縮）。
* **M3**：IPFS 等の分散 CAS、オフライン/マルチピア、ルート別ストリーミング SSR。

---

## 10. 運用 Tips（落とし穴と対策）

* **ハッシュ安定性**：ビルドパイプライン固定＋正規化のゴールデンテストを CI で回す。
* **WASM 粒度**：巨大1本より小分割（機能別）。
* **Action の純度**：UI が使う props を**サーバ側で確定**して返す（CSR/SSR 差異を縮小）。
* **再検証戦略**：`revalidateSeconds` をルート単位で設定、`props@hash` の差し替えで更新。
* **一致検証**：Server/Client の VNode ハッシュを比較し、差異を DevTools に可視化。

---

## 11. 一言サマリ

* **densha** は「**Git で動く UI**」：

  * 配信は **内容アドレス**、
  * 状態は **DAG** で可逆、
  * 副作用は **能力**で最小権限化、
  * SSR は **決定性**に限り導入——三方良しの運用を目指す。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/com-junkawasaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
