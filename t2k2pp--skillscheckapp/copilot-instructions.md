## skillscheckapp

> 実用性の高いC++開発で必要な知識を問う追加問題候補です。

# C++ スキルチェック問題 - 追加候補

実用性の高いC++開発で必要な知識を問う追加問題候補です。

## 基礎レベル (Beginner)

### 問題B1: 範囲ベースforループでの型指定
```
text: '範囲ベースforループで `std::vector<std::string>` の各要素を変更せずに処理する場合、最も適切な型指定はどれですか？'
options: [
  'for (auto item : vec)',
  'for (auto& item : vec)', 
  'for (const auto& item : vec)',
  'for (const auto item : vec)'
]
correctAnswerIndex: 2
explanation: 'const auto& を使うことで、不要なコピーを防ぎつつ、要素の変更を防げます。autoのみだとコピーが発生し、auto&だと変更可能になってしまいます。'
```

### 問題B2: 初期化リストによるメンバ変数初期化
```
text: 'クラスのconstメンバ変数を初期化するために必須な方法はどれですか？'
options: [
  'コンストラクタ本体で代入',
  '初期化リストで初期化', 
  'setterメソッドで設定',
  'デフォルト値で自動初期化'
]
correctAnswerIndex: 1
explanation: 'constメンバ変数は一度初期化されると変更できないため、コンストラクタの初期化リスト（: member(value) の形式）で初期化する必要があります。'
```

### 問題B3: ヘッダーファイルと実装ファイルの役割
```
text: 'ヘッダーファイル(.h)に書くべき内容として最も適切なものはどれですか？'
options: [
  '関数の実装コード',
  'クラスの宣言と関数のプロトタイプ',
  'グローバル変数の定義',
  'staticな実装の詳細'
]
correctAnswerIndex: 1
explanation: 'ヘッダーファイルには宣言（declaration）を書き、実装（definition）は.cppファイルに書きます。これにより、複数のファイルから同じ宣言を参照でき、リンク時の重複定義エラーを防げます。'
```

## 中級レベル (Intermediate)

### 問題I1: std::string_view の用途
```
text: 'C++17で導入された `std::string_view` の主な利点は何ですか？'
options: [
  '文字列を変更可能にする',
  '文字列の所有権を管理する',
  '文字列のコピーを避けて読み取り専用アクセスを提供する',
  '文字列の動的メモリ確保を自動化する'
]
correctAnswerIndex: 2
explanation: 'std::string_viewは文字列データのビュー（参照）を提供し、コピーを行わずに読み取り専用アクセスが可能です。関数引数でstd::stringの代わりに使うことで、パフォーマンスを向上できます。'
```

### 問題I2: emplace_back vs push_back
```
text: '`std::vector<std::pair<int, std::string>>` に要素を追加する際、最も効率的な方法はどれですか？'
options: [
  'vec.push_back(std::pair<int, std::string>(1, "hello"));',
  'vec.push_back({1, "hello"});',
  'vec.emplace_back(1, "hello");',
  'vec.insert(vec.end(), {1, "hello"});'
]
correctAnswerIndex: 2
explanation: 'emplace_backは引数を直接コンストラクタに転送し、コンテナ内で直接オブジェクトを構築します。push_backは一時オブジェクトを作成してからムーブするため、emplace_backの方が効率的です。'
```

### 問題I3: 初期化の違い
```
text: '`std::vector<int> v{10};` と `std::vector<int> v(10);` の違いは何ですか？'
options: [
  '両方とも同じ結果になる',
  '前者は要素10を1つ持つベクタ、後者は要素0を10個持つベクタ',
  '前者は要素0を10個持つベクタ、後者は要素10を1つ持つベクタ',
  '前者はコンパイルエラーになる'
]
correctAnswerIndex: 1
explanation: '波括弧{}は初期化リスト構築を行い、丸括弧()は通常のコンストラクタ呼び出しです。v{10}は値10を持つ要素1個、v(10)はデフォルト値0を持つ要素10個のベクタになります。'
```

### 問題I4: overrideキーワードの重要性
```
text: '仮想関数をオーバーライドする際に `override` キーワードを使用する主な理由は何ですか？'
options: [
  'パフォーマンスを向上させるため',
  'メモリ使用量を削減するため',
  'オーバーライドのつもりが新しい関数を定義してしまうミスを防ぐため',
  'アクセス指定子を変更するため'
]
correctAnswerIndex: 2
explanation: 'overrideキーワードにより、コンパイラが基底クラスの仮想関数を正しくオーバーライドしているかチェックします。シグネチャの違いなどでオーバーライドに失敗していた場合、コンパイルエラーで教えてくれます。'
```

## 上級レベル (Advanced)

### 問題A1: std::lock_guardの使用方法
```
text: '複数スレッドから共有リソースを安全にアクセスするための `std::lock_guard` の正しい使用方法はどれですか？'
options: [
  'std::lock_guard<std::mutex> lock(mutex); の後でmutex.lock();',
  'std::lock_guard<std::mutex> lock(mutex); だけで自動的にロック・アンロック',
  'mutex.lock(); std::lock_guard<std::mutex> lock(mutex);',
  'std::lock_guard<std::mutex> lock; lock.acquire(mutex);'
]
correctAnswerIndex: 1
explanation: 'std::lock_guardはRAII原則に従い、コンストラクタでmutexをロックし、デストラクタでアンロックします。スコープを抜ける際に自動的にアンロックされるため、例外安全です。'
```

### 問題A2: メモリオーダーの基本
```
text: '`std::atomic<int>` 変数に対する最もストリクト（厳格）なメモリオーダーはどれですか？'
options: [
  'std::memory_order_relaxed',
  'std::memory_order_acquire',
  'std::memory_order_seq_cst',
  'std::memory_order_consume'
]
correctAnswerIndex: 2
explanation: 'std::memory_order_seq_cst（sequential consistency）は最も厳格なメモリオーダーで、全スレッド間で一貫した順序を保証します。デフォルトでもあり、パフォーマンスより正確性を重視する場合に使用します。'
```

### 問題A3: 完全転送（Perfect Forwarding）
```
text: 'テンプレート関数で引数を別の関数に「完全転送」するための正しいコードはどれですか？'
options: [
  'other_func(args);',
  'other_func(std::move(args));',
  'other_func(std::forward<Args>(args));',
  'other_func(std::forward<Args...>(args...));'
]
correctAnswerIndex: 3
explanation: '可変長テンプレートでの完全転送では、std::forward<Args>(args)...の形式を使用します。これにより引数の値カテゴリ（左辺値/右辺値）を保持したまま転送できます。'
```

## 実用コード問題

### 問題C1: RAII クラスの実装
```
text: 'ファイルハンドルを安全に管理するRAIIクラスの基本構造として最も適切なものはどれですか？'
options: [
  'コンストラクタでファイルを開き、明示的なclose()メソッドで閉じる',
  'コンストラクタでファイルを開き、デストラクタで閉じ、コピーを禁止する',
  'コンストラクタでファイルを開き、デストラクタで閉じ、コピー可能にする',
  'ファイルを開くopenメソッドと閉じるcloseメソッドを提供する'
]
correctAnswerIndex: 1
explanation: 'RAIIでは、リソースの確保を初期化時、解放をデストラクタで行います。ファイルハンドルのような排他的リソースはコピーを禁止し、必要に応じてムーブセマンティクスを実装します。'
```

---

*これらの問題候補は、実際のC++開発でよく遭遇する実用的な知識を問う内容に焦点を当てています。*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/t2k2pp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/t2k2pp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
