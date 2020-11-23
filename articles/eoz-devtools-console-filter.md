---
title: 出力結果のフィルタリング on Chrome DevTools Console
emoji: 🍭
type: tech
topics: [devtools, EveOneZenn]
published: true
---

# はじめに

この記事は #EveOneZenn (Everyday One Zenn) の１日目です。

Chrome DevTools の Console における、出力結果のフィルタリングについてまとめます。

# ログレベルでフィルタリング

![](https://storage.googleapis.com/zenn-user-upload/04s6zcwu8f8de6luj0kq9u4pho6n)

Console パネル上部のドロップダウンから表示したいログのレベルを指定できます。

ここでは、 `Verbose`、`Info`、`Warnings`、`Errors` の４つのログレベルから表示したいログのレベルをそれぞれ指定します。
デフォルトでは `Verbose` を除く全てのログが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/lhkmp1nky7uceoogcf8k98yifqjx)

## ログレベルとログの種類

各ログレベルが指すログの種類は次のとおりです。

### `Verbose`

詳細レベルのログ。

`console.debug` で出力されたログが対象です。
また、パフォーマンスやセオリーに反するコードを実行した際にブラウザが出力する `[Violation]` アラートも `Verbose` に属するログです。

### `Info`

情報レベルのログ。

主に `console.warn` と `console.error` を除くすべてのログが対象です。

### `Warnings`

警告レベルのログ。

`console.warn` で出力されたログが対象です。

### `Errors`

エラーレベルのログ。

`console.error` で出力されたログが対象です。

### いずれにも属さないログ

下記のメソッドは上記のいずれのログレベルにも属さず、ログレベルでフィルタリングできません。

* `console.clear`
* `console.group`
* `console.groupCollapsed`

`console.group` と `console.groupCollapsed` は、グループ内で出力されたログはフィルタリングされますが、見出しはフィルタリングされません。

# テキストでフィルタリング

![](https://storage.googleapis.com/zenn-user-upload/44wc71c6nile5vzod0eualpc1ofh)

Console パネル上部の入力欄から表示したいログに含まれるテキストを指定できます。

また、テキストを `/[expression]/` とすることで、正規表現も使用できます。

例） `GET:`、`/(GET|POST):/`

なお、グルーピングされたログの見出しはテキストにマッチしていなくてもフィルタリングされません。

# メッセージソースでフィルタリング

![](https://storage.googleapis.com/zenn-user-upload/ds223fxdpi1lcaod2297v7os77e5)

Console パネル上部の ![](https://storage.googleapis.com/zenn-user-upload/m1i6p888ux3cnjb5a4copji1cz20) をクリックするとログを出力したファイルの一覧が表示されます。
ファイルを選択すると、そのファイルから出力されたログのみを表示できます。

最上段のセクションはすべてのログを、上から２番目のセクションは Console パネルでの操作で出力されたログを、それ以外は各ログレベルに対応したログを出力したファイルを表示します。

同じファイルからのログを追跡したいときに便利です。

# 参考

* [Get Started With Logging Messages In The Console  |  Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/console/log)
* [Console API Reference  |  Chrome DevTools  |  Google Developers](https://developers.google.com/web/tools/chrome-devtools/console/api)
