## get-md

> URL をブラウザで取得し、指定要素を Markdown に変換する CLI ツール。

# get-md

URL をブラウザで取得し、指定要素を Markdown に変換する CLI ツール。

## Tech Stack

- Rust (Edition 2024)
- headless_chrome (CDP 経由のブラウザ制御)
- htmd (HTML -> Markdown 変換、skip_tags/spacing オプション使用)
- clap (CLI 引数解析、derive feature)
- indicatif (プログレス表示)
- regex (日時パターンマッチング)
- url (相対URL -> 絶対URL 変換)
- anyhow (エラーハンドリング)

## Architecture

- WebDriver を使用せず、システムの Chrome/Chromium を CDP で直接制御
- JS レンダリング対応（SPA、動的コンテンツ）
- `--no-cache` で CDP の `Network.setCacheDisabled` によりブラウザキャッシュを無効化
- CSS セレクタでブラウザ内 JS 実行により要素の outerHTML を取得
- htmd で HTML -> Markdown 変換（script, style, noscript, svg は skip_tags で除去）
- Rust 側で相対 URL を絶対パスに変換（Markdown リンク・画像の `[text](url)` パターン、`[text](<url>)` 形式、`\(` `\)` を含むリンク先の解析、`<...>` 内の `)` を終端として誤認しない解析、`<...>` 内の `\>` をエスケープとして正しく処理、通常リンク先URL内のクォート保持、`\ ` を含む通常リンク先の解析に対応。インラインコードとコードフェンス内は変換せず、未閉鎖のインラインバッククォートはリテラルとして扱い、実際のリンク/画像構文ではない単独の `](` は無視する）
- テーブルのセルパディングとセパレータダッシュを圧縮（コードフェンス内は変更しない）。セル内のエスケープ済みパイプ（`\|`）は区切りとして扱わない
- 対応 OS: macOS, Windows

## Project Structure

```
src/
  main.rs       # CLI 定義、ブラウザ起動、HTML 取得、Markdown 変換
  progress.rs   # indicatif ベースのプログレス表示
Makefile        # build, release, test, fmt, check, install ターゲット
.github/
  workflows/
    ci.yml      # CI (test, clippy, fmt, build)
    release.yml # リリース (バージョンバンプ、ビルド、GitHub Release、Homebrew更新)
```

## Development

```bash
make build    # デバッグビルド
make release  # リリースビルド
make test     # テスト
make check    # clippy + check
make fmt      # フォーマット
make install  # /usr/local/bin にインストール
```

## Testing

- ユニットテストは Chrome 不要（CLI パース、JS エスケープ、Markdown変換、URL解決、リンクパーサー、コード領域を除外する URL 解決、未閉鎖バッククォートを含むリンク解決、フェンスマーカー検出、テーブルセル分割、ファイルステータス判定（実フロー再現含む）、日時無視比較、エスケープ判定、開き括弧検出、リンク候補検出、インラインコード閉じ検出、マルチバイト文字対応、フェンスコード直後のリンク解決、インラインコードとリンクの混在、プログレス表示、CRLF正規化、ネストフェンスのスキップ、画像リンク検出、リンク先分割の境界条件、連続テーブル圧縮、連続山括弧リンク解決、URL解決のエラー分岐、タイトル内エスケープ引用符、未閉鎖タイトル、ネスト括弧内の引用符、CRLFフェンスブロック、コロンのみセパレータ、行途中開始のインラインコード検出、テーブル行中央揃え・幅広パディング圧縮、タイムアウト境界値、ファイルステータス等値比較、ネストフェンス長さ不一致、多段相対パス解決、複数リンク候補検出、山括弧リンク先フォールバック、4段ネスト括弧、日時除去単独日付、無効日付パターン、日付バウンダリ境界、深ネスト引用符、altなし画像URL解決、双方非UTF-8日時比較、タブパディング圧縮、ブロッククォート内リンク解決、時刻のみパターン除外、標準・山括弧混在リンクのテスト）
- E2E テストは実際の Chrome/Chromium が必要（`#[ignore]` 付き）。GitHub Raw の実取得、ローカル `file://` ページでの相対 URL 解決、複数セレクタの `---` 結合、`--ignore-date` の書き込み抑止を確認する
- `make test` または `cargo test` で実行

## Key Design Decisions

- Chrome/Chromium はシステムにインストール済みであることを前提とする
- セレクタ未指定時は body 全体を対象とする
- 複数セレクタ指定時は `---` で区切って結合
- ファイル出力時は末尾改行を保証
- 完了表示は出力書き込み成功後にのみ表示する（✨ created / 📝 updated / ✔ unchanged）
- ファイルステータス判定: 新規→created、内容変更→updated、同一内容→unchanged。git管理下で未ステージ変更がある場合は常にupdated。既存ファイルの読み取りに失敗した場合はupdated。ファイルの存在状態は `File::create` 前に記録し、書き込み後の `path.exists()` に依存しない。git 判定は対象パスに最も近い既存ディレクトリを起点に行い、削除済みの tracked ファイルや repo 外 cwd からの実行でも契約を守る
- `--ignore-date`: 日時パターン（`YYYY-MM-DD HH:MM(:SS)?`、スラッシュ区切り、`Z`・小数秒・タイムゾーン付き ISO 8601）を無視してファイル比較し、日時だけの差分なら上書きせず unchanged 扱いにする。双方に日時パターンを含む場合のみ比較し、非 UTF-8 や日時パターンを含まない場合は安全のため通常比較にフォールバックする。git 管理下の未ステージ変更がある場合は `file_status` と同じ契約で updated 扱い
- `idle_browser_timeout` は `timeout + 30s` のバッファを saturating 加算で設定する
- バージョニングは CalVer (YY.M.counter) 形式

---
> Source: [owayo/get-md](https://github.com/owayo/get-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
