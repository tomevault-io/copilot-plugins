## filepathstudio

> このプロジェクトでは、WPF で次のアプリケーションを作成します。

# AGENTS.md

## 目的

このプロジェクトでは、WPF で次のアプリケーションを作成します。

- 大量のファイルパスをテキストファイルで管理出来るアプリ。

- テキストエディタでファイルパス情報を含んだファイルを編集出来る。

- テキストエディタにおいて、ファイルパスを書いた行の横にインラインで「Open in Exploler」ボタンがある (ボタンでなくても化。クリック可能テキストラベルのほうが実装しやすいならそれで進める)。これにより、ファイルを簡単にエクスプローラーで開くことができる。

- 追々「Open in Code」ボタンなども実装する。

- 編集中のテキストにおいて、コメント部分はシンタックスハイライトされ、地の文 (ファイルパス) は別色で表示されるようにしたい。ファイルパスにおいて、スラッシュ、バックスラッシュはまた色を分けたい。

- ウィンドウ最上部にはメニューバーがある。「File」「Terminal」(タイトルバーの代わりにメニューバーがウィンドウ左上にある)
   - File: "Open File", "Save File", "Save File As...", "Exit"
   - Terminal: "Open Terminal" (編集中ファイルの保存ディレクトリを別ウィンドウでターミナルを開ける)
   ui:TitleBarButton, ui:TitleBarButton.Content を使って、タイトルバー部分にメニューバーを配置しつつ、Height="{Binding ElementName=Window, Path=(ui:TitleBar.Height)}" なども必要に応じて用いて。恐らく、タイトルバーのテキストが邪魔になるので空欄で良さそう。

- タブ機能があり、複数のファイルを編集できる。

## アプリケーションが扱うファイル形式

基本的に地の文はファイルパスです。# がついている行はコメントです。

例:
```
# これはコメントです
C:\Users\user\Documents\file.txt
C:\Users\user\Documents\file2.txt

# これはコメントです (2)
C:\Users\user\Documents\file3.txt
```

## プロジェクト構造

- 主な使用ライブラリは ModernWpfUI, AvalonEdit です。

- アプリケーションのプロジェクトは `FilepathStudio` です。

- ビルドは `dotnet build` で出来ると思われます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sashi0034)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sashi0034)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
