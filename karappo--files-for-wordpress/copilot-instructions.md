## files-for-wordpress

> WordPress テーマ共通ユーティリティ。

# karappo-common

WordPress テーマ共通ユーティリティ。

## サブディレクトリインストール時の注意

WordPress を `/wp/` 等のサブディレクトリにインストールし、サイトURLをルートに設定している構成では、`wp-config.php` に以下を追加すること:

```php
define('COOKIEPATH', '/');
```

### 理由

- `project-specific.php` の `rest_authentication_errors` フィルターが未ログインユーザーのREST APIアクセスを制限している
- サブディレクトリ構成ではログインCookieのパスがデフォルトで `/wp/` になるため、`/wp-json/` へのリクエストにCookieが送られない
- その結果、ログイン中でもGutenbergエディタからの保存が403エラーになる
- `COOKIEPATH` はテーマファイルからは設定不可（WordPress初期化前に定義が必要）なため、`wp-config.php` に書く必要がある

---
> Source: [karappo/files-for-wordpress](https://github.com/karappo/files-for-wordpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
