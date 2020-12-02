---
title: ESLint で warning が存在する場合にもチェックを Failure させる
emoji: 🍭
type: tech
topics: [eslint, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の10日目です。

ESLint でチェック結果に warning が存在する場合、チェック自体を Failure させる方法を紹介します。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-rscss-in-css-modules

# --max-warnings オプション

ESLint の CLI には `--max-warnings` というオプションがあります。
これは、チェック中に存在して良い warning の上限を指定するものです。

たとえば、次のようなコマンドで ESLint を実行した場合、 warning は 3 件まで許容されます。

```sh
$ eslint --ext .js --max-warnings 3 .
```

このオプションの初期値は `-1` （上限なし）です。

見出しの通り、ひとつでも warning がある場合にチェックを Failure させたい場合は、オプションの値は `0` とします。

```sh
$ eslint --ext .js --max-warnings 0 .
```

これで、 error もしくは warning が１件でもあれば Failure する厳密なチェックが実行できます。

# おまけ： warnings をすべて無視する

ESLint で warning をレポートさせないようにするには `--quiet` オプションを使います。

次のコマンドで ESLint を実行した場合、 warning はレポートされず、 error のみレポートされます。

```sh
$ eslint --ext .js --quiet .
```

# 参考

* [Command Line Interface - ESLint - Pluggable JavaScript linter](https://eslint.org/docs/user-guide/command-line-interface)
