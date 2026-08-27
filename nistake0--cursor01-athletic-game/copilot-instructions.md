## test-aaa-sut

> テストコードは AAA パターンと sut 命名に従うルール


# テストコードの書き方（AAA パターン・sut）

このプロジェクトでは、**単体テスト・統合テストを書くとき**に次のルールに従う。

## 1. AAA パターン

テストは **Arrange（準備）・Act（実行）・Assert（検証）** の 3 フェーズで構成する。

- **Arrange**: モックの用意、テスト対象の生成、前提条件の設定。
- **Act**: テスト対象のメソッド呼び出しや、検証したい操作を 1 つ実行する。
- **Assert**: 期待する値・状態・呼び出しを検証する。

## 2. フェーズを示すコメント（必須）

各フェーズの**ブロックの直前にコメントを書く**。接頭辞でどのフェーズか分かるようにする。

- **Arrange**: `// Arrange.` または `// Arrange: 〜`（何を準備したか短く書いてもよい）
- **Act**: `// Act.` または `// Act: 〜`
- **Assert**: `// Assert.` または `// Assert: 〜`

例:

```ts
it('zone の gravityFactor が重力に反映される', () => {
  // Arrange: 空中・zone に gravityFactor 0.5 を返すモック
  setupGameMock({ getGroundY: () => 600, getCurrentZone: () => ({ gravityFactor: 0.5 }) });
  const sut = new PlayerManager(mockApp, mockGame as any, eventEmitter);
  sut.initializePlayer();
  mockPlayer.x = 100;
  mockPlayer.y = 400;

  // Act: 1 フレーム update
  sut.update();

  // Assert: velocityY の増分が GRAVITY * 0.5 であること
  expect(sut.getVelocityY()).toBe(PLAYER.GRAVITY * 0.5);
});
```

## 3. テスト対象オブジェクトの名前は sut（必須）

**テストの主たる対象となるオブジェクト**（クラスのインスタンスや、テスト対象の関数を呼ぶ主体）には、変数名 **`sut`**（Subject Under Test）を使う。

- 例: `const sut = new PlayerManager(...);` のあと、`sut.update()`, `sut.getVelocityY()` のように呼ぶ。
- 複数の対象がいる場合は、**主に検証の対象になる 1 つ**を `sut` にし、他は役割が分かる名前（例: `mockGame`, `eventEmitter`）のままでもよい。

## 4. コメントで「何をモックしているか」「何を検証しているか」を書く

- **Arrange**: どのモックを用意し、どの値を返すようにしたか（または何を設定したか）が分かるようにする。
- **Assert**: どの値・どのメソッド呼び出しを、どの期待値で検証しているかを書く。

一瞥してテストの意図と検証内容が把握できるようにする。

## 5. 既存テストの修正

既存の `*.test.ts` / `*.spec.ts` を編集するときも、上記ルールに合わせてコメントの追加・sut へのリネームを行う。

---
> Source: [nistake0/cursor01-athletic-game](https://github.com/nistake0/cursor01-athletic-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
