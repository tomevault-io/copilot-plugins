## noslop

> このリポジトリで作業する AI エージェント (と人間の開発者) 向けの案内。利用者向けの説明は [README.md](README.md) にある (日本語だけで書く。英語版は置かない)。

# noslop — エージェント向けガイド

このリポジトリで作業する AI エージェント (と人間の開発者) 向けの案内。利用者向けの説明は [README.md](README.md) にある (日本語だけで書く。英語版は置かない)。

## プロジェクト概要

noslop は、日本語の文章から「AI 臭さ」を機械的に拾う Rust 製の Linter。判定器ではなく、疑わしい箇所を決定的に並べ、直すかどうかは書き手に委ねる。

- 文字種・語句パターン・文長の統計で判定する。品詞で数えるルール (P15・P16・R18・R20) は hasami の辞書で判定する (`morph.rs`)。既定 (`dictionary = "auto"`) では、share ディレクトリに取得した配布辞書を hasami の推奨順で選び、なければバイナリに同梱した IPAdic の辞書 (`dict/ipadic.hsd`) を使う。元の校正は同梱の IPAdic で行ったので、その条件で判定するなら `dictionary = "bundled"`。`--no-dict` や、同梱しないビルドで辞書が見つからないときは辞書なしの近似で動く
- 設定はユーザーの設定 (`~/.config/noslop/config.toml`) とプロジェクトの設定 (`noslop.toml`) の 2 層で、既定値 < ユーザー < プロジェクト < CLI の順に項目ごとに重ねる (`config.rs`)
- 文書 (Markdown・テキスト) のほかに、コードのコメント・静的な文字列・JSX/HTML の本文と表示属性 (tree-sitter で取り出す。`code/`) を検査できる。コメントや表計算のセルのような互いに独立した短い断片の集まり (`DocumentKind::Fragments`) には、1 文ずつ判定するルール (`RuleUnit::Sentence`) だけを当てる。文書・段落をまたいで数えるルールは、ひと続きの地の文で校正したため
- 既定で有効にするのは、コーパスで誤検知率を確かめた語句と閾値だけ。未校正のものは experimental にする
- AI 臭さ (`slop`) と読みやすさ (`readability`) の 2 つのレーンを混ぜない
- 文書全体の点数は出さない。指摘を 1 件ずつ並べ、レーンと重大度ごとの件数を数えるだけにする (以前の「自然度スコア」は校正しておらず、人の文書と AI の文書を見分けていなかったので外した。[点数を出さない理由](docs/revision.md#文書全体の点数を出さない理由))

## 技術スタック

- Rust (edition 2024)。ツールチェーンの版は `mise.toml` が正
- Markdown: `pulldown-cmark`
- 文分割と形態素解析: [hasami](https://github.com/owayo/hasami)。文分割は `hasami::sentence` (辞書を使わない)、形態素解析は `analyzer` feature (辞書があるときだけ)。crates.io の同名クレートは別物なので、git の依存でリリースのタグ (`Cargo.toml` の `tag`) を指定して入れる。テストは `build` feature (dev-dependency) で小さな辞書を組み立てる。既定の feature `bundled-dict` で `dict/ipadic.hsd` を `hasami::include_hsd!` でバイナリに埋め込み、`Dictionary::from_static` で複製せずに読む。辞書を指定しないとき (`auto`) は、share ディレクトリに取得した配布辞書を同梱の辞書より先に使う (選ぶ順は依存の hasami の推奨順で、目録の自動更新では変わらない)。辞書を差し替える手順は `dict/README.md`。上げるときはタグを書き換えて `cargo update -p hasami` を実行し (hasami の `rust-version` が上がっていれば、`Cargo.toml` の `rust-version` もそろえる。上げると clippy が MSRV で抑えていた指摘を出すことがある)、例外表の版を固定したテスト (`segment.rs`) が落ちたら分割の差分と THIRD_PARTY_NOTICES.md の NOTICE の写しを確かめる。依存の更新で HSD の形式が変わるときは、配布辞書の目録 (`dict/catalog.json`) と同梱の辞書も同じ形式に更新する。目録の辞書の形式 (`format_version`) が依存の `hasami::hsd::FORMAT_VERSION` と同じことはテスト (`the_catalog_is_readable_by_the_linked_hasami`) が確かめる。同梱の辞書を上げる手順は `dict/README.md` にある (テスト `the_bundled_dictionary_matches_dict_readme` が `dict/ipadic.hsd` を `dict/README.md` の表の SHA-256 と照らす)
- 配布辞書の目録: `dict/catalog.json` (hasami のリリースに添付された `dictionaries.json` をそのまま置く) が正本。`build.rs` が serde・serde_json (build-dependencies) で読んで検証し、`HASAMI_TAG`・`DEFAULT_SOURCE`・`CATALOG_FORMAT_VERSION`・`RECOMMENDED`・`DICTIONARIES` を作る。`src/dictionaries.rs` はそれを `include!` で取り込む (壊れた目録ではビルドが止まる)。目録は Release のワークフローが hasami の最新のリリースに合わせて自動で更新する。手で更新するなら `make dict-catalog` (`TAG=...` で版を指定、省くと最新。`tools/dict-catalog.sh` が gh で取得し、古い版には戻さない)、目録の辞書を実際に取得できるかは `make dict-check` (通信が要る) で確かめる。依存の hasami (`Cargo.toml` のタグ) と同梱の辞書は判定の結果を左右するので、目録とは別に人が上げる
- コードのコメントの取り出しと、gws のコマンドの解析: `tree-sitter` (0.27) と言語ごとの文法の crate。文法はすべて常にバイナリに入れる (feature で分けない。どの配布物でも同じ言語を読めるように。バイナリの大きさは、同梱辞書とターゲットによって変わる)。Bash の文法は gws のコマンドの解析にも使う。Ruby の文法は astro-sight と同じく owayo のフォーク (git の依存) を使う。SQL の文法 (tree-sitter-sequel) は `cc` を `~1.2` に固定させ、`ring` が引く新しい `cc` とぶつかるので入れない。文法の crate は tree-sitter 0.27 で読める版に固定している (上げるときは文法ごとに ABI の対応を確かめる。0.27 の API の変更は tree-sitter-guide スキル)。文法は配布のバイナリに静的にリンクするので、THIRD_PARTY_NOTICES.md の「tree-sitter と言語の文法」の表に版と著作権の表示を載せている。文法を足す・上げるときは表も直す
- 語句の照合: `aho-corasick` / `regex`
- ファイル探索: `ignore` (.gitignore を尊重)、並列化: `rayon`
- CLI: `clap`、設定: `toml` + `serde`、出力の色: `anstream` / `anstyle`
- 配布辞書の取得 (`noslop dict download`): hasami の feature `download` (`hasami::download`) に任せる。HTTP は hasami が引く `ureq` (TLS は rustls。社内の CA を入れた環境でも通るよう、OS の証明書ストアで検証する `platform-verifier`)、圧縮版 (`.hsd.zst`) の展開は `ruzstd`、検証は `sha2`、置き換えは `tempfile`。noslop は目録の値と取得元を渡し、誤りを日本語に言い換える。取得の細部 (受信・展開・照合・一時ファイル) のテストは hasami にあり、noslop は目録の渡し方・言い換え・CLI を確かめる

## 構成

```mermaid
flowchart TD
    CLI[cli.rs<br/>引数・サブコマンド] --> CFG[config.rs<br/>ユーザーの設定と noslop.toml の読み込み・重ね合わせ]
    CLI --> CHANGED[changed.rs<br/>git の差分・変更行との重なり]
    HOOK --> CHANGED
    CLI --> WALK[walk.rs<br/>対象ファイルの列挙]
    WALK --> DOC[document.rs<br/>Document / Block / Sentence]
    DOC --> MD[markdown.rs<br/>Markdown → ブロック]
    DOC --> TXT[plaintext.rs<br/>テキスト → ブロック]
    DOC --> CODE[code/<br/>コードの文言 → ブロック]
    MD --> DIR[directive.rs<br/>抑制コメントの読み取り]
    TXT --> DIR
    DOC --> SEG[segment.rs<br/>文分割]
    CFG --> ENG[engine.rs<br/>ルールの選択・実行]
    DOC --> ENG
    ENG --> RULES[rules/<br/>phrases・rhythm・structure・custom]
    ENG --> MORPH[morph.rs<br/>形態素解析 (辞書は同梱)]
    RULES --> MORPH
    ENG --> SUP[suppress.rs<br/>抑制の適用]
    ENG --> OUT[output/<br/>text・json・toon・github・brief]
    CLI --> DIFF[diff/<br/>改稿の前後の比較]
    CLI --> CAL[calibrate.rs<br/>コーパスでの校正]
    CLI --> MCP[mcp.rs<br/>MCP サーバー]
    CLI --> HOOK[hook/<br/>Claude Code・claw-hooks のフック]
    HOOK --> GWS[gws/<br/>gws で書き込む値の取り出し]
    CLI --> DICTS[dictionaries.rs<br/>配布辞書の取得]
    MORPH --> DICTS
    DIFF --> ENG
    CAL --> ENG
    MCP --> ENG
    HOOK --> ENG
    MCP --> OUT
    HOOK --> OUT
```

## モジュールの責務

| ファイル | 責務 |
|---------|------|
| `src/main.rs` | エントリポイント。終了コードを返す |
| `src/cli.rs` | clap の定義と、`check` / `diff` / `calibrate` / `rules` / `explain` / `init` / `mcp` / `hook` / `skill-install` / `dict` の実行。引数の誤りは clap の既定の 2 で終えるが、フック (`hook git-diff` を除く) は 1 にする (`parse_error_code`。Claude Code と claw-hooks は 2 を「止める」と読むので、設定の書き誤りで作業を止めないように)。`check` と `diff` はルールの選び方の引数 (`EngineArgs`) を共有する。`check --git-diff` は同じ作業ツリーの変更を選び、全体を検査した後に変更行へ絞る。パス省略は作業ツリー全体、指定時は変更との交差。`check` は出す内容 (`--report full\|brief`) と形式 (`--format`) の組み合わせを最初に確かめる。設定はユーザーの設定とプロジェクトの設定の 2 層で読み (`load_config_from`。`--config` はプロジェクトの層だけを差し替え、`--no-config` はどちらも読まない)、項目ごとに重ねてエンジンの設定 (`config_engine_options`) と対象の列挙の設定 (`walk_options`) に組み立てる (CLI が優先)。書いた項目だけを上書きし、配列 (`[files] extensions`・`exclude`) は丸ごと置き換える。`[morphology] dictionary` の相対パスは、書いたファイルのディレクトリ基準で直してから重ねる。`[[custom]]` は ID (大文字小文字を区別しない) でまとめ、同じ ID はプロジェクトの定義で丸ごと置き換える。手元の環境 (ホームディレクトリ、`HASAMI_DICT`、share ディレクトリ) は `Environment` にまとめ、入口 (`check`・`diff`・`calibrate`・`rules`・MCP・フック) で 1 度だけ作って渡す (テストでは空の `Environment` を渡す) |
| `src/config.rs` | 設定ファイルの書式 (`ConfigFile`) と読み込み。ユーザーの設定の置き場所 (`user_config_path`。`~/.config/noslop/config.toml`。ホームは `std::env::home_dir()` で、Windows は `USERPROFILE`。`XDG_CONFIG_HOME` は見ず、自動では作らない。ひな形は `noslop init --user` が `user_template()` を書く。`noslop init` の `template()` と同じく、`[files]` と `[code]` の `extensions` に、読める拡張子すべて (`walk::DEFAULT_EXTENSIONS` と `CodeLanguage` の表から作る) をコメントにして並べる) と、プロジェクトの設定の探索 (`discover`。`noslop.toml` / `.noslop.toml` をカレントから親へ探し、最初の 1 つ)。読んだ 2 層は `ConfigLayers` にまとめ、項目の値はプロジェクト、なければユーザーの順に取る (`pick`)。書式の誤りはどちらの層でも終了コード 2 |
| `src/changed.rs` | `check --git-diff` とフックで使う git の差分の列挙 (`WorkTree`)・行から原文の範囲への変換・指摘と文脈の重なり (`touches`)。変更範囲 (`ChangeScope`) は全文・既知の変更行・範囲不明を区別し、全文専用ルールの実行条件と検査後の絞り込みを `ChangeScope::lint` で共有する。差分は一度の解析で、指摘の絞り込みに使う行と、全文判定に使う実際の追加・置換行を収集する。削除の隣接行は絞り込みだけに使う。対象の選択も共有する |
| `src/walk.rs` | 対象ファイルの列挙 (.gitignore・.ignore・.noslopignore・拡張子・設定の除外。拡張子は文書の `[files] extensions` とコードの `[code] extensions` のどちらかに当たれば集める (`WalkOptions::selects`。コードの拡張子は既定で空で、コードの拡張子でないものは `cli::walk_options` が誤りにする)。除外は .gitignore と同じ書式で、プロジェクトの設定の除外は設定ファイルのディレクトリが基準、ユーザーの設定の除外は検査の起点 (`check` に渡したディレクトリ) が基準。直接指定したファイルは拡張子と除外を問わない) |
| `src/engine.rs` | ルールの選択 (stable / experimental / ジャンル / 明示の有効化・無効化)、設定値の適用、実行、重大度の上書き、fingerprint、並べ替え。`[rules] enable` / `disable` と `[rules.X]` は、ユーザーとプロジェクトの層ごとに ID に直してから、ルールごとに CLI > プロジェクト > ユーザー > 既定で決める (同じ層では無効が優先。同じ層で ID と名前の両方から同じルールを指せばエラー)。`[rules.X]` の severity と閾値はキーごとにプロジェクトが上書きする。辞書を使う有効なルール (`Rule::uses_morphology`) があるときだけ辞書を読み、文書ごとに `DocMorphology` を作ってルールに渡す。断片の集まり (`DocumentKind::Fragments`) には、`Rule::unit` が `RuleUnit::Sentence` のルールだけを当てる。`Coverage::Partial` の文書では `Rule::requires_full_document` が真のルールを実行しない (明示的な有効化よりも優先) |
| `src/morph.rs` | 形態素解析。`[morphology]`・`--dict`・`--no-dict` から辞書を決めて読み込む (`resolve`)。辞書の指定は `auto` (既定)・`bundled` (同梱の辞書)・`share:<名前>`・ファイルのパスで、`auto`・`bundled` は文字列が正確に一致したときだけキーワードとみなす (その名前のファイルは `./auto` と書く)。`auto` の探す順は、`HASAMI_DICT` → share ディレクトリの配布辞書 (`dictionaries::preferred_in`。依存の hasami の推奨順 `hasami::analyzer::DISTRIBUTED_DICTS` で、ない候補だけを飛ばす。配布辞書でない `*.hsd` は選ばない) → 同梱の辞書 (同梱しないビルドでは share ディレクトリのほかの `*.hsd` を名前順に探し、`auto` で見つからなければ辞書なし、`required` ならエラー)。`share:<名前>` は share ディレクトリの辞書に直す (`HASAMI_DICT` には当てない)。見つかった辞書・指定した辞書が読めなければエラーにし、次の候補や同梱の辞書に切り替えない。`auto` の選び方は読み込みと分けてある (`auto_pick`。`dict list` と `dict download` が、指定しないときに使う辞書を示すのにも使う)。share ディレクトリと `HASAMI_DICT` の値は呼び出し側から渡し (`MorphologyOptions::search`。既定は空で手元を探さない)、in-process のテストで環境変数を触らずに済むようにしている。同梱の辞書は埋め込んだバイト列を複製せずに読み (`Dictionary::from_static`)、組み立てはプロセスで 1 度だけにして共有する (`Morphology::bundled`)。文書ごとの解析器で、ルールが求めた文だけを解析して覚えておく。使った方式 (`MorphologyStatus`) は出力の `settings` に載る |
| `src/suppress.rs` | 抑制コメントを診断に当てる。未知のルール名は警告にする |
| `src/output/` | text (色付き。ファイルの中をレーンごとの節に分け、要約もレーンごとに数える)・json (安定スキーマ)・toon (JSON と同じデータを TOON で。符号化は `toon-format` クレート)・github (ワークフローコマンド、エスケープは出力器の責務)・brief (AI や編集者に渡す改稿指示。`Brief` のデータを組み立て、Markdown・JSON・TOON に描き分ける。データはルールの表と該当箇所の表に分け、TOON の表形式が効くようにしている) |
| `src/diff/` | `noslop diff`。指摘の突き合わせ (`mod.rs`、fingerprint を多重集合で)、事実の消失と追加 (`facts.rs`)、改稿の偏り (`shifts.rs`)、出力 (`render.rs`) |
| `src/calibrate.rs` | `noslop calibrate`。人の文書と生成文書で、ルールごとの誤検知率・検出率、`Rule::measure` による閾値の掃引、昇格候補と見直しを出す (手順は `docs/calibration.md`) |
| `src/mcp.rs` | `noslop mcp`。標準入出力の JSON-RPC で `check` / `diff` / `explain` / `rules` を提供する。`check` は `report` (既定 `brief`) と `format` (`markdown`・`json`・`toon`) で返すものを選ぶ (版の扱いは `docs/integrations.md`) |
| `src/hook/` | エージェントのフック。`claude_code.rs` は `noslop hook claude-code` で、フックの入力 (JSON) をイベントで振り分ける (PostToolUse の Write / Edit / MultiEdit はこのファイル、PreToolUse の Bash は `gws.rs`、Stop は `stop.rs`。`hook_event_name` がなければ PostToolUse とみなす)。PostToolUse は入力から変わった行を求め、重なる指摘だけを短い brief で返す。`file.rs` は `noslop hook file` で、パスだけを渡すフックの仕組み (claw-hooks の extension_hooks など) 向け。変わった行を git の差分 (HEAD との比較。追跡していないファイルと git の外はファイル全体。`changed.rs`) から求め、同じ brief をテキストで返す (`--max-chars` で行単位に切る)。`stop.rs` は Stop と、フックの入力を渡せない Stop の仕組み (claw-hooks の stop_hooks など) 向けの `noslop hook git-diff`。どちらも作業ディレクトリを含む git の作業ツリー (`changed.rs` の `WorkTree`) の、コミットしていない変更 (HEAD との差分の行と、追跡していないファイルの全体) に重なる指摘を返す。git は index を書き換えないように呼ぶ (`diff.autoRefreshIndex=false`。並んで動く自動コミットと index.lock を取り合わないため)。`gws.rs` は gws で書き込む値の検査で、PreToolUse (Bash) の入口 (`crate::gws` でコマンド行から取り出す) と、2 つの入口で共有する検査と結論 (`review_writes`。入口の違いは `Request` と `Style` で渡す) を置く。本文 (ドキュメントの本文) は値ごとに文章の文書に、短い値 (セル・タイトル・置き換えの文字列) は呼び出しごとに断片の集まりの文書にし、文書の名前にはコマンドと値の決まる宛先を入れる。本文に警告以上の指摘がある書き込みは 1 度だけ止め (`deny`)、検査した書き込みごとに記録を置く。同じ書き込み (セッション・コマンド・`--dry-run` か・宛先・値) は、止めてから 30 分のあいだ検査せずに通す (通しても記録を消さず、期間も延ばさない。claw-hooks は呼び出しごとに判定して最初に止めたところで打ち切るので、通すたびに消すと、止める書き込みが 2 つあるコマンドがいつまでも通らない)。記録はキャッシュの置き場所 (`cli::Environment::cache_dir`) にハッシュと時刻だけを置き、名前の材料に記録の形の版 (`RECORD_FORMAT`) を入れる。記録を残せないときは止めない。短い値・情報だけ・`--dry-run` の指摘と、呼び出しの形が実行時に変わりうる書き込み (`gws::Write::exact` が偽) は止めずに知らせる。gws を含まない Bash は、解析も設定の読み込みもしない。`command.rs` は `noslop hook command` で、claw-hooks のコマンドフックの判定器。claw-hooks が渡す gws の呼び出し 1 つの引数 (JSON。プロトコルの版 1) を `gws::Arg` に直し (値が決まり 1 語なら `Static`、値が決まらない 1 語は `Unknown`、`zero_or_more` は `Dynamic`)、`gws::write` と `review_writes` で検査する。止めるなら理由を標準エラーに書いて終了コード 2、知らせるなら標準出力に書いて 0、入力や設定の誤りは 1。`analysis` が `uncertain` なら止めず、`context_delivery` が偽か `PermissionRequest` なら知らせない。名乗りは claw-hooks が `[noslop]` を付けるので付けない。`mod.rs` は共通の部品で、設定を 1 度だけ読んで対象の選び方とエンジンを持つ `Reviewer` (`selects`・`lint_file`・`brief`) と、1 ファイルの検査 (`review`。変更範囲に応じた検査と絞り込みは `ChangeScope::lint` に任せる)・行単位の切り詰め (`truncate_lines`) を置く |
| `src/code/` | コードのコメントと静的な文言を、文章として検査するブロックにする (tree-sitter)。構文木は入口で一度作り、コメントと `static_text/` で共有する。静的な文字列の復号は `static_text/decode.rs`、言語ごとの境界は `strings.rs`、JSX/HTML の本文・表示属性は `markup.rs`。`[code] static_text` は既定で真、偽ならコメント/docstring だけ。補間・独立した属性や操作部品は連結しない。通常区間は `TextMap::push_exact`、復号した一件は `push_opaque` で元の範囲を保ち、全て断片として扱う。`mod.rs` は言語 (`CodeLanguage`。拡張子) と入口 (`parse`)。`extract.rs` は言語ごとの文法でコメントのノード (と Python のモジュール・クラス・関数の先頭の docstring) を取り出し、普通のコメントかドキュメントのコメント (`///`・`/** */` など) かを分ける。`body.rs` は記号 (`//`・`#`・`/*`・行頭の `*` など) を外し、行ごとに外した幅を記録する。日本語を含まないコメント・shebang・ツールへの指示 (`eslint-disable`・`noqa` など)・著作権とライセンスの表記は外す。隣り合う行の同じ種類・同じ列の行コメントは 1 つにまとめ、コードの後ろのコメントは単独にする。`build.rs` は本文をブロックにする。普通のコメントは段落 (記号の行はリスト項目) として組み、改行は既定で文の区切りにする (句点を打たない 1 行 1 文のコメントが多く、つなぐと長い一文として数えるため。前の行が読点・助詞・開き括弧で終わる・次の行が閉じ括弧・読点・注記の丸括弧で始まる・英文の折り返しのときだけつなぐ)。ドキュメントのコメントは Markdown として読んで (`TextMap::compose` で原文の位置に戻す)、Javadoc と C# の XML ドキュメントはタグを外して読む。コメントの中身全体が抑制の記法 (`noslop-disable-next-line P01 -- 理由`、`<!-- -->` で囲んでもよい) なら抑制として読む (`directive::parse_code_comment`)。テストは `tests.rs` (言語ごとの取り出し・まとめ方・位置・抑制) |
| `src/gws/` | gws (Google Workspace CLI) のコマンドから、Google ドキュメント・スプレッドシートに書き込む値を取り出す。`shell.rs` は Bash のコマンド行を tree-sitter-bash で読み、gws の呼び出しの引数を実行せずに読む (クォート・`$'...'`・`"$(cat <<'EOF' … EOF)"` のヒアドキュメント。`cat`・`/bin/cat`・`/usr/bin/cat` だけを本文をそのまま出すコマンドとして扱い、任意のパスの同名ファイルは扱わない。変数・コマンド置換・展開を含む引数は値が決まらないものにし、展開がクォートの中だけなら 1 語 (`Arg::Unknown`)、クォートしていない展開・グロブを含むなら消えることも分かれることもある語 (`Arg::Dynamic`) にする)。`mod.rs` はサービス・リソース・メソッドの表 (`METHODS`) で、書き込む値の場所 (フラグと、JSON の中のパス) と扱い (本文か短い値か) を引く。表にないコマンドは読まない。値の決まらない語が、値を取るフラグの値の位置に 1 語でしかなければ、呼び出しの形は決まる (`Write::exact`。フックが書き込みを止めてよいかの判断に使う)。コマンドを実行したり、コマンドが指すファイルを読んだりはしない |
| `src/skill.rs`・`skills/SKILL.md` | `noslop skill-install`。`skills/SKILL.md` をバイナリに埋め込み、`~/.claude/skills/noslop/` か `~/.codex/skills/noslop/` に書く。`make install` もバイナリを入れたあとに両方へ入れる (`SKILL_TARGETS` で選ぶ)。CLI の使い方を変えたら SKILL.md も直す (本文に `$` の直後の数字や `$ARGUMENTS` を書かない。スキルの引数に置き換わる) |
| `src/dictionaries.rs` | `noslop dict download` / `list`。hasami の配布辞書の表 (`DICTIONARIES`。名前・大きさ・SHA-256) と取得元 (`HASAMI_TAG`・`DEFAULT_SOURCE`。目録の版の GitHub のリリースの添付ファイル) は、`build.rs` が `dict/catalog.json` から作ったものを `include!` で取り込む。表示の日本語の説明は `Distributed::description` (名前ごと。知らない名前は目録の説明)。share ディレクトリ (`share_dir`。hasami の `hasami::analyzer::data_dir` をそのまま使う。`HASAMI_DATA_DIR` → `$XDG_DATA_HOME/hasami` → Windows は `%LOCALAPPDATA%\hasami` → `~/.local/share/hasami`) と、`share:<名前>` の解決 (`resolve_share`。名前にパスの区切り・`:`・`..` を書かせない)。取得は毎回実行し、既存の辞書も検証後に置き換える。`hasami::download::download` に `force: true` で任せる (`download`。表の項目を `DistributedDict` に直し、取得元は目録の版の `DEFAULT_SOURCE` を必ず渡す。目録は依存の hasami より新しいことがあるため)。既定では圧縮版 (`<名前>.hsd.zst`) を受け取りながら展開し、圧縮版と展開後の両方の大きさ・SHA-256 と、辞書として読めることを確かめてから rename で置く (失敗しても既存のファイルは消さず、壊さない。24 時間より古い一時ファイルは次の取得で消す)。圧縮版が HTTP 404 のときだけ非圧縮版へ自動で切り替える。通信・検証・展開の失敗では切り替えない。`--uncompressed` で最初から非圧縮版を取得できる。hasami の誤り (`DownloadError`。non_exhaustive) はバリアントごとに日本語に言い換え (`DownloadError::from_hasami`)、展開したものの誤り (出所が `<URL> (decompressed)`) は「展開した中身」と書く。置き場所の検証 (`check_file`) も `hasami::download::verify`。取得した辞書は、辞書を指定しないとき (`auto`) に次の実行から使われる (`preferred_in` が依存の hasami の推奨順で選ぶので、目録の自動更新では選ぶ順が変わらない)。`dict list` は、指定しないときに使う辞書を示す |
| `build.rs` | 配布辞書の目録 (`dict/catalog.json`) を検証し (版・形式の版・名前・ファイル名・大きさ・SHA-256 の書式・推奨の辞書・圧縮版 (`compressed`) のファイル名 `<名前>.hsd.zst`・大きさ・SHA-256)、`src/dictionaries.rs` が取り込む定数を `OUT_DIR/catalog.rs` に作る。辞書は小さい順に並べる。目録の形式の版が依存の hasami で読めるかは、build.rs から依存を参照できないので `src/dictionaries.rs` のテストで確かめる |
| `src/heading.rs` | 見出しの形 (コロン型・問い型・番号型) の分類。S07 と `noslop diff` で共有する |
| `src/document.rs` | 文書モデル。解析用テキストと原文の対応 (`TextMap`。別の文字列の上で作った対応を原文の位置に重ねる `compose` もある)、行・列 (`LineIndex`)。形式 (`SourceFormat`。拡張子から Markdown・テキスト・コード (`Code(CodeLanguage)`) を決める) と、文書の組み立て (`DocumentKind`。ひと続きの文章 `Prose` か、コメントやセルのような断片の集まり `Fragments` か。コードは `Fragments`) |
| `src/markdown.rs` | Markdown をブロック (段落・リスト項目・見出し・表セル) に分け、コード・URL・装飾を解析用テキストから外す。front matter は文書の先頭のものだけを自前で見つけて解析から外す (pulldown-cmark のメタデータブロックの記法は文書の途中の `---` にも当たって本文を捨てるので、有効にしない)。段落の中の改行のうち、書式から文の区切りと分かるものは `Block::sentence_breaks` として記録する (`breaks_sentence`。1 行 1 項目の箇条書きやラベルの行を 1 文につながないため)。次の行が Markdown の記法にない箇条書きの記号・番号 (「・」「◯」「①」「(1)」) か「短い見出し＋全角コロン」で始まる、直前の行がコロンで終わる・【】で囲んだ見出し風の行・太字だけの行である、直前の行が日本語を含まない英文の文末で次の行が日本語で始まる、ハード改行の直前が文末記号・読点・ひらがなで終わらない、直前の行が丁寧体の文末 (「です」「ます」「ください」など) で終わり次の行が括弧で始まらない、のどれか。ひらがなや読点で終わる行の改行は本文の折り返しとみなしてつなぐ |
| `src/plaintext.rs` | テキストをブロックに分ける (空行・字下げ・箇条書き記号。空行がほとんどない文書は 1 行 1 段落とみなし、文末記号のない短い 1 行は見出しと推定する。コメントだけの行は段落を切らない) |
| `src/segment.rs` | 文分割。`hasami::sentence` への橋渡し (括弧の対応を取ってから、対応の取れた括弧の内側と例外表の語の内側では分割しない)。改行を文の区切りにするモードでは、`line_breaks` の位置を改行とみなして分割する (`split_with_breaks`)。書式から文の区切りと分かる改行 (`sentence_breaks`) は、改行の扱いの設定によらず切る。組み込みの例外表の版 (`BUILTIN_EXCEPTIONS_VERSION`) をテストで固定し、表が変わったら気付けるようにしている |
| `src/directive.rs` | `<!-- noslop-... -->` の読み取りと、コードのコメントの中身全体に書いた抑制の記法の読み取り (`parse_code_comment`)。記法の本体の読み方は 2 つで共有する |
| `src/text.rs` | 文字種の判定と文長の数え方。文末記号・括弧の判定は、文分割と字の集合がずれないよう `hasami::sentence` のものを再公開する。文末の記号を除いた本体と、名詞らしい終止 (体言止め) の推定もここに置き、R06・R12 と `noslop diff` で共有する |
| `src/genre.rs` | ジャンルと別名 |
| `src/diagnostic.rs` | 診断・重大度・レーン・ステータス・原文上の範囲 |
| `src/rules/mod.rs` | `Rule` trait・`RuleMeta`・`Scope`・`builtin_rules`、判定の単位 (`RuleUnit`。1 文ずつか、文書・段落をまたぐか)、全文が対象のときだけ実行する条件 (`Rule::requires_full_document`)、校正用の測定値 (`Measure`・`Fires`) と校正の基準の重大度 (`calibration_basis`) |
| `src/rules/phrases.rs` | 語句パターン系 (`P`)。`phrases/catalog.rs` が語句辞書で動く P01〜P12・P18・P19、`phrases/syntax.rs` が構文の型 (P13・P14・P20)、`phrases/attribution.rs` が出典をぼかした権威付け (P21)、`phrases/artifacts.rs` が引用マーカーの残骸 (P22)、`phrases/markup.rs` が太字にならずに記号のまま残った強調の記法 (P23。語句ルールのスコープの設定に従わず、見出し・表・引用を含む全ブロックの 1 行の中で `**…**` の対を探す。Markdown は描画されない対を警告、テキスト・コード・gws の値は出力先で描画されるか分からないので情報)、`phrases/effect.rs` が効果をぼかす言い回し (P24。語句ごとに文脈を見直す実験的なルール)、`phrases/stance.rs` が出典の態度の解説 (P25。主語から 40 字以内の述語を、主語のあとで開いた括弧の外で探す)、`phrases/jargon.rs` が行動をぼかす業界語 (P26。業務の対象との組み合わせに絞る)、`phrases/closer.rs` が結びの定型 (P27。P18 の結びを分け、途中の段落も含めて言い換えを拾う)、`phrases/permission.rs` が許可の形で語る行動 (P28。状態・選択肢・質問などを除く文字列の近似。必要な許可は残せる)、`phrases/reading.rs` が読みやすさのルール (P15〜P17)、`phrases/engine.rs` が照合の共通部品 |
| `src/rules/rhythm.rs` | リズム・統計系 (`R`)。ルールごとに `rhythm/` 配下のファイル (burstiness・endings・length・buried_list・antithesis・paragraphs・leads・cleft・self_answer・overcorrection・commas・duplicates・future_closer・triads・guide_density・nominal_lists・action_closer・style) |
| `src/rules/structure.rs` | 構造系 (`S01`〜`S10`) |
| `src/rules/custom.rs` | 設定ファイルの独自ルール (`[[custom]]`) |
| `src/rules/testing.rs` | ルールのテスト用の近道 (`run` / `run_with` / `matched`) と、`measure` と `check` の一致の確認 (`assert_measures_agree`) |

位置は内部で原文の UTF-8 バイト位置 (`Span`) に統一し、行・列は出力時に計算する。解析用テキストの位置は `Block::to_source` で原文に戻す。

## ルールを足す・変えるとき

1. 系統を決める。文の中の表現なら `P`、文や段落をまたぐ集計なら `R`、Markdown の体裁なら `S`
2. ID はその系統の次の番号にする。**公開した ID の意味は変えない**。廃止したルールの ID は再利用しない
3. `RuleMeta` を書く。`lane`・`status`・`default_severity`・`summary` と、下の書式の `explanation`
4. 語句ルールは語句ごとにステータスと重大度を持たせる。人間の文章にも一定数出る語は `info` にする。実験的な項目でも人の文章によく出るものは、`with_note(WEAK_SIGNAL_NOTE)` で弱い手掛かりの注記を付ける。前後の文字を条件にだけ使う正規表現は、指す範囲を名前付きグループ `m` で囲む (`(?:^|[^が])(?P<m>…)`。Rust の regex は前後の読みを使えないため。`phrases/engine.rs` の `MATCH_GROUP`)。重なる一致は先に始まる長いほうが残るので、校正済みの項目と同じ位置から始まる、より長い実験的な項目を作らない (校正済みの重大度を上書きしてしまう)
5. テストを書く。検出する例・検出しない例・スコープ (段落だけか)・experimental の有効化・原文上の位置 (`matched` で原文の文字列と一致すること) の 5 点は必ず押さえる
6. 閾値を持つルールは `Rule::measure` を実装し、`noslop calibrate` で閾値を掃引できるようにする。実装したら `rhythm.rs` / `structure.rs` のテストにある `MEASURED` に ID を足す。`testing::assert_measures_agree` が、測定値が閾値を越えることと `check` が指摘することの一致を、閾値を測定値の前後に動かして確かめる (指摘する例と、値は測れるが指摘しない例を 1 つ以上用意する)。重大度を切り替えるだけの閾値 (R05 の `error_above` など) は `Measure::at` で切り替え先の重大度を示す
7. 特定の重大度の率で校正したルールは、`Rule::calibration_basis` でその重大度を返す (既定は、既定の重大度が警告以上なら警告、情報なら情報。R05 は重大)。`noslop calibrate` は、見直しの判定をこの重大度以上の指摘で数える
8. 品詞で判定したほうが元の校正条件に近いルールは、`Rule::uses_morphology` を真にし、`ctx.morph` (辞書があるときだけ `Some`) の形態素で判定する。辞書がない・文を解析できないときは辞書なしの近似に戻す。テストは `morph::testing::morphology` で小さな辞書を組み立て、`testing::run_with_morphology` で当てる (辞書なしと辞書ありの結果が違う例を並べる)
9. 1 文の中だけで判定が決まるルールは、`Rule::unit` で `RuleUnit::Sentence` を返す (コードのコメントや表計算のセルのような断片の集まりにも当たる)。文をまたぐ・数を数える・文書の構造を見るルールは既定 (`RuleUnit::Document`) のままにする (断片をまとめた母数で数えることになるため)
10. docs/rule-catalog.md のルール一覧を更新し、`make docs` で `docs/rules.md` を作り直す

### `explanation` の書式

`noslop explain` と `docs/rules.md` にそのまま出る。見出しは次の 5 つで固定する。

```markdown
### 何を見るか
(検出の条件を具体的に。閾値があれば数値で)

### なぜ問題か
(なぜ AI 臭さ・読みにくさにつながるのか)

### 直し方
(どう直すか。機械的に全部直す必要はないことも書く)

### 例
- 直す前: ...
- 直した後: ...

### 根拠
(校正の結果、または未校正で experimental である理由)
```

例文は自分で作る。既存の資料の文面をそのまま写さない。

## 校正の原則

- **閾値はデータなしに変えない**。変えるなら、人間の文書での誤検知率と AI の文書での検出率を `noslop calibrate` で測り (手順は [docs/calibration.md](docs/calibration.md))、根拠を `explanation` の「根拠」に書く。コーパスはリポジトリに入れない
- **未校正のものは experimental にする**。辞書なしの近似で元の校正条件から外れるものも同じ
- **辞書ありの判定は元の校正条件 (品詞で数える) に合わせる**。辞書あり・なしで結果が変わるルールは、それぞれの検出条件・制約と校正の結果を `explanation` の「根拠」に書く。利用者向けのドキュメントとその生成元の説明文には、過去の検出器への言及や比較を載せず、現在の noslop の動作を説明する。辞書で精度が上がらないルール (R06 など) は辞書を使わない。既定の `auto` は手元の share ディレクトリの辞書で変わるので、`noslop calibrate` で測るときは `dictionary = "bundled"` (校正した条件) か、測りたい辞書を設定ファイルで固定する
- **文書単位の指標を足すなら、先に検証する**。ルールごとの誤検知率の校正は、指摘を足し合わせた値が文書を見分けることを保証しない。文書全体の点数や判定を出すなら、ジャンルと長さを分けた保留のデータで見分けられることを示してから、目的と名前を決める
- **統計系ルールは地の文だけで集計する**。校正を地の文 (見出し・リスト・引用・表・コードを除く) で行ったため
- **語句ルールの既定のスコープは段落だけ**。リスト・表・引用は設定で広げる
- 人間のほうが多く使うと分かった語は外す。人間にも一定数ある語は `info` に下げる

## テストとチェック

開発のコマンドは Makefile にまとめてある (一覧は `make help`)。make は `mise.toml` の版のツールを `mise exec` 経由で呼ぶので、シェルで mise を activate していなくてよい。

```bash
make setup                         # mise.toml の Rust を入れ、依存を取得する (初回と依存の更新後)
make ci                            # CI と同じ検査 (fmt・clippy -D warnings・test・docs-check・同梱しないビルドの test)
make test                          # テストだけ
mise exec -- cargo test segment::  # モジュールを絞る
make docs                          # docs/rules.md を作り直す
```

- push 前に `make ci` を通す。make が `mise exec` 経由で動くので、手元と CI で clippy の版がずれない。cargo のコマンドには `--locked` が付く (`Cargo.lock` を更新したいときは `CARGO_FLAGS=` で外す)
- CI は ubuntu / macos / windows で検査を回す。Linux と macOS のジョブは `make setup` と `make ci` だけを呼ぶので、検査を足すときは Makefile の `ci` に足し、`.github/workflows/ci.yml` に検査のコマンドを並べない。Windows の build ジョブはランナーの make (mingw32-make) を避け、`docs-check` 以外の同じ検査を cargo で直接呼ぶ。`ci` を変えたら、ci.yml の build ジョブの Windows のステップもそろえる。Windows では改行を LF のまま checkout している (テストの行・列・バイト位置は LF 前提。`.gitattributes` でも LF に固定している)
- `make ci` は `docs/rules.md` を生成し直して差分がないことも確かめる (`make docs-check`。ファイルは書き換えない)。ルールの定義や説明文を変えたら `make docs` を忘れない
- 統合テストは `tests/cli.rs` (サブコマンドの入出力・終了コード) と `tests/integrations.rs` (MCP サーバーとフック) にある。組み込みルールの増減で壊れないよう、件数は設定ファイルの独自ルールと `--only-rules` で確かめる
- 手元の設定と辞書にテストが左右されないようにする。統合テストの補助関数 (`noslop()`) は、`HOME`・`USERPROFILE`・`HASAMI_DATA_DIR` を空の一時ディレクトリにし、`HASAMI_DICT` を外して、手元のユーザーの設定 (`~/.config/noslop/config.toml`) と share ディレクトリの辞書から切り離す (Windows の `std::env::home_dir()` は `HOME` ではなく `USERPROFILE` を見るので、両方を渡す)。ユーザーの設定や share ディレクトリを使うテストは、その変数を上書きして一時ディレクトリに置く。in-process のテストは環境変数を書き換えず (Rust 2024 では `set_var` が unsafe で、並列のテストとも競合する)、ホームディレクトリと share ディレクトリを引数で渡す
- 配布辞書の目録 (`dict/catalog.json`) から来る値 (辞書の名前・大きさ・SHA-256・版) をテストに直書きしない。`noslop::dictionaries` の `DICTIONARIES`・`RECOMMENDED`・`HASAMI_TAG` から組み立てる (Release のジョブが目録を更新したときに、テストで止まらないように)。取得できたときの動きは単体テスト (`dictionaries`・`cli`) で、目録の実物を取得して読めることは `make dict-check` で確かめる
- テスト用の文章は、実在の文書や既存の資料を写さずに自分で書く
- 辞書ありの判定のテストは、既定の探索や同梱の辞書に頼らず、hasami の `DictBuilder` で小さな辞書を組み立てる (`morph::testing`、`tests/cli.rs` の `write_dictionary`)。ただし小さな辞書は hasami の既定の文字種で動くので、全角空白 (U+3000) がトークンにならないなど配布の IPAdic と違う点がある。全角空白のトークンに依る処理は形態素を手で組んで確かめる。share ディレクトリの探索を確かめるテストは、組み立てた辞書を配布辞書の名前 (`ipadic-neologd-sudachi.hsd` など) で一時ディレクトリに置き、`HASAMI_DATA_DIR` でそこを指す (`HASAMI_DATA_DIR` は `XDG_DATA_HOME` より優先される)。同梱の辞書に依るテストは `cfg(feature = "bundled-dict")` で分ける。同梱しないビルドのテスト (`make test-no-default-features`。中身は `cargo test --no-default-features`) は `make ci` に入っている
- コードのコメントの文法はすべて常に入るので、言語ごとのテストを feature で分けない。新しい言語を足したら、文法を読み込んでコメントを 1 つ読めること (`every_grammar_loads_and_reads_a_comment`) に加える。コメントの位置のテストは、解析用テキストの日本語の字が原文の同じ字に戻ることまで確かめる
- Rust の文字列の行継続 (`\`) は次の行の先頭の空白を消す。説明文を数字や `(` の前で折り返すときは、`\` の前に空白を入れる (「90 字台から 100 字」のように、数字の前後の空白が落ちるため)

## 利用者に見せる文言

- 辞書や配布バイナリのサイズをドキュメント・コメント・テストに固定値で書かない。配布辞書のサイズは目録 (`dict/catalog.json`) を正本とし、利用者には `noslop dict list` での確認を案内する。同梱辞書の同一性は SHA-256 で確かめる
- 出力・ヘルプ・ルールの説明文・ドキュメントでは、レーンを「AI 臭さ」「読みやすさ」「独自ルール」(`Lane::label_ja`) と呼ぶ。内部の ID (`slop` / `readability` / `custom`)、JSON のフィールド、オプション名 (`--no-readability` など) はそのまま使う
- 1 件ずつの指摘は、どのレーンでも「指摘」と呼ぶ。数えるときは「読みやすさの指摘 13 件」のように「〜の指摘」を付ける (「読みやすさ 13 件」だけでは、読みやすい点が 13 あるという良い評価にも読める)。レーンやルールそのものを指すときは「読みやすさのレーン」「読みやすさのルール」のように「〜の」でつなぐ
- 「指さし」「読解負荷」のような別の呼び方を作らない。出力ごとに呼び名が違うと、要約の件数と一覧の指摘が対応しなくなる

## 公開リポジトリとしての注意

- 社内の組織名・チーム名、実在のメールアドレス、ホームディレクトリの絶対パスを、コード・ドキュメント・テスト・コミットメッセージに書かない。例には `you@example.com` や `~` を使う
- ルールの知見をどの情報から得たか (資料・記事・論文・Web ページの名前と URL や、どの調査・ツール・検出器から得たかの説明) は、コード・ドキュメント・説明文・テスト・コミットメッセージに書かない。説明文の「根拠」には、検出の条件と noslop のコーパスで測った結果だけを書く。[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) に載せるのは、ライセンスが求める著作権表示と許諾文だけ。資料の文面をそのまま持ち込まない
- 図は Mermaid で書く (ASCII アートは使わない)
- コメントとドキュメントは日本語で書く。利用者に見せる診断の文も日本語

## リリース

GitHub Actions の Release ワークフロー (workflow_dispatch) で行う。版は日本時間で `YY.M.COUNTER` (例: `26.9.100`) で、同じ月の 2 回目以降は COUNTER を 1 ずつ上げる。`dry_run` では版と配布辞書の目録を確認し、コミット・タグ・配布物のビルド・公開は行わない (目録が変われば検査は回る)。成果物は Linux x86_64 / arm64 (arm64 は `ubuntu-24.04-arm` のランナーでそのままビルドする)、macOS x86_64 / arm64、Windows x86_64 のアーカイブと `SHA256SUMS`。アーカイブは depup・astro-sight と同じ `noslop-<ターゲット>.tar.gz` (Windows は `.zip`) で、最上位にバイナリと `LICENSE`・`THIRD_PARTY_NOTICES.md` を置く (同梱の辞書 IPAdic (NAIST-2003) と tree-sitter の文法 (MIT) は、再配布に表示を添えることを求めるため)。

公開の後、`update-homebrew` のジョブが Homebrew の tap (`owayo/homebrew-noslop`) の `Formula/noslop.rb` を、公開したリリースの `SHA256SUMS` の値で書き直して push する (formula は depup の tap と同じく、OS と CPU ごとにリリースのアーカイブを指し、`bin.install "noslop"` と `prefix.install "THIRD_PARTY_NOTICES.md"` で入れる。LICENSE は Homebrew が自動で入れる。bottle は作らない。書いた formula は `ruby -c` で確かめてから push する)。tap への push は GitHub App のトークンで行い、Variables の `APP_CLIENT_ID` と Secrets の `PRIVATE_KEY` が要る。どちらかがなければ警告を出してこのジョブだけを飛ばす (リリースは落とさない)。添付の名前 (`noslop-<ターゲット>.tar.gz`) を変えるときは、このジョブの formula のテンプレートと README のインストールの表も作り直す。

Release のワークフローは、先に `dictionary-catalog` のジョブ (読み取りの権限だけ) で `make dict-catalog` を回し、配布辞書の目録を hasami の最新のリリースに合わせる。目録が変わったときだけ、このジョブで `make ci` と `make dict-check` を通し、`prepare-release` が版の更新と同じコミットに `dict/catalog.json` を入れる。`GITHUB_TOKEN` で push したコミットでは ci.yml が走らないので、目録の検査はこのジョブで完結させている。dry_run でも目録のジョブは回り、差分の表示に `dict/catalog.json` が入る。

mise 自身は `ci.yml` と `release.yml` の mise-action が、公開から 14 日以上たった版を選ぶ (`minimum_release_age: 14d`)。mise-action のキャッシュは無効にし (`cache: false`)、CI のビルドのキャッシュだけを rust-cache に任せる。リリースではキャッシュを使わない。

---
> Source: [owayo/noslop](https://github.com/owayo/noslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
