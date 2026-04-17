## practice-kyoto-souvenir

> Ionic routing and tabs implementation rules based on official documentation


## Ionic Router Outlet Rules

### ion-tabs の仕様

**CRITICAL**: `ion-tabs`コンポーネントは、Ionicの標準仕様として、内部で自動的に`ion-router-outlet`として機能します。

> "The ion-tabs component does not have any styling and works as a router outlet in order to handle navigation."
> 
> — [Ionic Framework Documentation: ion-tabs API](https://ionicframework.com/docs/api/tabs)

> "The ion-tabs component renders another ion-router-outlet which is responsible for rendering the contents of each tab."
> 
> — [Ionic Framework Documentation: Angular Navigation - Working with Tabs](https://ionicframework.com/docs/angular/navigation#working-with-tabs)

### 実装ルール

- **`tabs.page.html`内に明示的に`ion-router-outlet`を追加してはいけない**
  - `ion-tabs`が自動的にrouter outletとして機能するため、明示的な追加は不要
  - 明示的に追加すると、`ion-tabs`が自動生成する`ion-router-outlet`と**同じ階層で重複**し、ルーティングが競合する

- **階層構造**:
  ```
  app.component.html
  └── ion-router-outlet (メイン: /tabs, /sights/:id, /souvenir/:id をレンダリング)
      └── tabs.page.html (ion-tabs)
          └── ion-router-outlet (自動生成: /tabs/discover, /tabs/favorites, /tabs/plans をレンダリング)
  ```

### タブの動作原理

> "With Tabs, the Angular Router provides Ionic the mechanism to know what components should be loaded, but the heavy lifting is actually done by the tabs component."
> 
> — [Ionic Framework Documentation: Angular Navigation - Working with Tabs](https://ionicframework.com/docs/angular/navigation#working-with-tabs)

Angular Routerはどのコンポーネントをロードすべきかを決定しますが、実際のナビゲーション処理は`ion-tabs`コンポーネントが行います。

## Correct Implementation Pattern

### 正しい実装例

```html
<!-- app.component.html -->
<ion-app>
  <ion-router-outlet></ion-router-outlet>  <!-- メインのルーティングアウトレット -->
</ion-app>

<!-- tabs.page.html -->
<ion-tabs>
  <!-- ion-router-outletは追加しない（ion-tabsが自動的にrouter outletとして機能する） -->
  <ion-tab-bar slot="bottom">
    <ion-tab-button tab="discover" href="/tabs/discover">
      <ion-icon name="compass"></ion-icon>
      <ion-label>発見</ion-label>
    </ion-tab-button>
    <!-- ... -->
  </ion-tab-bar>
</ion-tabs>
```

### 誤った実装例

```html
<!-- ❌ 誤り: tabs.page.htmlにion-router-outletを追加してはいけない -->
<ion-tabs>
  <ion-router-outlet></ion-router-outlet>  <!-- これは追加してはいけない -->
  <ion-tab-bar slot="bottom">
    <!-- ... -->
  </ion-tab-bar>
</ion-tabs>
```

**問題**: 明示的に`ion-router-outlet`を追加すると、`ion-tabs`が自動生成する`ion-router-outlet`と**同じ階層で重複**し、ルーティングが競合してスクロールやリンクが機能しなくなります。

## 公式ドキュメント参照

- [Ionic Framework: ion-tabs API](https://ionicframework.com/docs/api/tabs)
- [Ionic Framework: Angular Navigation - Working with Tabs](https://ionicframework.com/docs/angular/navigation#working-with-tabs)
- [Ionic Framework: Angular Navigation - Nested Routes](https://ionicframework.com/docs/angular/navigation#nested-routes)

## Related Files

- `e2e/playwright/discover-page.spec.ts`: 再発防止テスト
- `src/app/app.component.html`: メインルーティングアウトレット
- `src/app/pages/tabs/tabs.page.html`: タブページ実装
- `INCIDENT_REPORT.md`: 詳細なインシデントレポート

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hideokamoto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
