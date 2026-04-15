## tsumugi

> `apps/desktop/app/hooks/` 配下の SWR hooks に関するルール。

# SWR Hooks ルール

`apps/desktop/app/hooks/` 配下の SWR hooks に関するルール。
リファレンス実装: `app/hooks/ai.ts`

## useAdapter() の使用制限

- `useAdapter()` は **hooks 内でのみ** 使用する
- コンポーネント（`.tsx`）から `useAdapter()` を直接呼ばない
- adapter の操作は必ず hooks を経由する

## SWR キー設計

- 各 hook ファイルで SWR キー用の `interface` を明示的に定義する
- fetcher / mutation 関数の第1引数からキーのプロパティを分割代入する
- 引数は必須パラメータ（`null` や `undefined` は使わない）
- ID が確定してからコンポーネントをマウントする（条件付きレンダリング）

```ts
interface PlotKey {
  type: 'plot';
  id: string;
}

export function usePlot(id: string) {
  const adapter = useAdapter();
  return useSWR<Plot | null, Error, PlotKey>(
    { type: 'plot', id },
    ({ id }) => adapter.plots.getById(id),
  );
}
```

## useSWRMutation の書き方

- `useSWRMutation` のキーは、再フェッチしたい `useSWR` のキーと一致させる
- mutation 関数の第1引数からキーのプロパティを分割代入して使う（外側の関数引数を直接参照しない）

```ts
export function useCreatePlot(projectId: string) {
  const adapter = useAdapter();
  return useSWRMutation<Plot, Error, PlotTreeKey, CreatePlotData>(
    { type: 'plotTree', projectId },
    ({ projectId }, { arg }) => adapter.plots.create(projectId, arg)
  );
}
```

### Data 型の使い分け: `null` vs `undefined`

- **`null`**: キャッシュを「データなし」に明示的に上書きする場合（`populateCache: true` と併用）
- **`undefined`**: 戻り値に関心がなく、revalidate に任せる場合（deleteFromTree, reorder など）
- `void` は SWR のジェネリック型引数に使えない（ESLint `no-invalid-void-type`）
- `undefined` を返す fetcher は `async` にして明示的に `return undefined` する（`Promise<void>` は `Promise<undefined>` に代入不可）

### delete の書き方

- 削除後はキャッシュを `null` で上書きし、再フェッチしない（`populateCache: true, revalidate: false`）
- 削除対象が存在しなくなるため、サーバーへの再フェッチは不要

```ts
export function useDeletePlot(id: string) {
  const adapter = useAdapter();
  return useSWRMutation<null, Error, PlotKey>(
    { type: 'plot', id },
    async ({ id }) => {
      await adapter.plots.delete(id);
      return null;
    },
    { populateCache: true, revalidate: false },
  );
}
```

### delete（ツリーキーにバインド）の書き方

- 削除対象（個別キー）とは異なるキー（ツリーキー）にバインドする場合、`populateCache` / `revalidate` は付けない
- 削除対象の ID は `trigger(id)` で渡す
- ツリーキーに対する通常の revalidate（再フェッチ）が走り、ツリー一覧が更新される

```ts
export function useDeletePlotFromTree(projectId: string) {
  const adapter = useAdapter();
  return useSWRMutation<undefined, Error, PlotTreeKey, string>(
    { type: 'plotTree', projectId },
    async (_, { arg: id }) => {
      await adapter.plots.delete(id);
      return undefined;
    },
  );
}
```

> `useDeletePlot(id)` は削除対象自身のキー（`PlotKey`）にバインドするため `populateCache: true, revalidate: false` でキャッシュを null に上書きする。
> `useDeletePlotFromTree(projectId)` はツリーキー（`PlotTreeKey`）にバインドするため、デフォルトの revalidate でツリー一覧を再フェッチする。

### reorder の書き方

- ツリーキーにバインドし、revalidate でツリー一覧を再フェッチする
- Data 型は `undefined`（戻り値に関心がない）

```ts
export function useReorderPlots(projectId: string) {
  const adapter = useAdapter();
  return useSWRMutation<undefined, Error, PlotTreeKey, ReorderPlotsArg>(
    { type: 'plotTree', projectId },
    async (_, { arg }) => {
      await adapter.plots.reorder(arg.parentId, arg.orderedIds);
      return undefined;
    },
  );
}
```

## JSDoc

- 全ての公開 hooks に JSDoc を書く
- `@param` で引数を説明する
- mutation hooks には `@revalidates` タグで、trigger 後に再フェッチされる hook を明記する

```ts
/**
 * AIセッションを作成し、初回メッセージを送信する
 * @param projectId - プロジェクトID
 * @revalidates useAISessions - AIセッション一覧を再フェッチする
 */
export function useCreateAISession(projectId: string) { ... }
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takecchi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
